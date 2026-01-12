MODIS数据下载

------

## 1  为什么文件名里查不到 “纬度>65°”

- **MOD02／MYD02**（Terra/Aqua）属于 **5 分钟 swath（扫描条带）级别**的 L1B 数据。
- **文件命名规则**（以 MOD021KM 为例）
   `MOD021KM.AYYYYDDD.HHMM.CCC.YYYYDDDHHMMSS.hdf`
  - `YYYYDDD` 观测年+年积日
  - `HHMM`  观测开始的 UTC 时间
  - `CCC`  版本号（Collection，现行 061）
  - **文件名里并不含空间范围信息**——你没法靠字符串判断它是否扫过极区。  ([MOD021KM - Level 1B Calibrated Radiances - 1km - LAADS DAAC](https://ladsweb.modaps.eosdis.nasa.gov/missions-and-measurements/products/MOD021KM?utm_source=chatgpt.com))

因此要“按纬度筛选”，只能靠 **元数据查询**（API）或事后自己读 MOD03/MYD03 的经纬度 SDS。推荐前者，因为可以在下载前就剔除无关 granule，节省带宽与存储。

------

## 2  两种官方 API：LAADS API-v2 与 CMR

下面两条思路都能一次性返回 **“只覆盖纬度 ≥ 65° 的 granule 列表＋下载链接”**，区别主要在接口风格。

|      | LAADS API-v2                                                 | CMR (Common Metadata Repository)                  |
| ---- | ------------------------------------------------------------ | ------------------------------------------------- |
| 优势 | ① 查询条件最丰富；② 能直接返回 **原始存储路径**，curl/wget 一步下载。 | ① 覆盖所有 DAAC；② 返回 JSON 更标准，生态脚本多。 |
| 劣势 | 必须先去 LAADS 网站申请 **download token**。                 | 需要先找出每个产品的 **collection_concept_id**。  |

> **极区两块查询范围**
>
> - 北极：`[BBOX]N90 S65 W-180 E180`
> - 南极：`[BBOX]N-65 S-90 W-180 E180`
>    （BBOX 顺序不敏感；这是 LAADS API 的写法，CMR 用 `bounding_box=-180,65,180,90`）

### 2.1  LAADS API-v2 核心用法

1. **申请 token**
    打开 [https://ladsweb.modaps.eosdis.nasa.gov](https://ladsweb.modaps.eosdis.nasa.gov/), 登录 Earthdata → 右上角 **Tools & Services → API-v2 → Generate Token**。保存一串 30 多字符的 token。  ([API-V2 User Guide - LAADS DAAC](https://ladsweb.modaps.eosdis.nasa.gov/tools-and-services/api-v2/user-guide/))
2. **拼查询 URL**（只查列表，不下载）

```
https://ladsweb.modaps.eosdis.nasa.gov/api/v2/content/details
 ?products=MOD021KM,MYD021KM          # 可换成 021KM/02HKM/02QKM
 &collections=61
 &temporalRanges=2017-01-01..2024-12-31
 &regions=[BBOX]N90 S65 W-180 E180
 &formats=json                       # 建议 json，默认也是 json
```

1. **一次搜一边极区**（把 regions 换成南极 BBOX 即可）。
2. **下载**
   - 直接把上面 `details` 换成 `archives`，就会返回纯文本文件清单；
   - 用 wget：

```bash
wget -e robots=off -m -np -R .html,.tmp \
     --header "Authorization: Bearer $TOKEN" \
     "https://ladsweb.modaps.eosdis.nasa.gov/api/v2/content/archives?产品...同上" \
     -P ./MODIS
```

接口参数说明见官方 **OpenAPI spec**，“regions” 一节详细列出了 `[BBOX]N… S…` 写法。  ([API-V2 OpenAPI Specifications - LAADS DAAC](https://ladsweb.modaps.eosdis.nasa.gov/tools-and-services/api-v2/specs/content))

### 2.2  CMR 查询思路

1. 找到 **collection_concept_id**
   - Terra MOD021KM： `C1378227407-LAADS`
   - Aqua MYD021KM： `C1378293238-LAADS`（可在 Earthdata Search → Collection 信息页 URL 里看到）。  ([MODIS/Terra Calibrated Radiances 5-Min L1B Swath 1km](https://cmr.earthdata.nasa.gov/search/concepts/C1378227407-LAADS.html?utm_source=chatgpt.com))
2. GET 请求示例（北极）：

```bash
curl -G https://cmr.earthdata.nasa.gov/search/granules.json \
     --data-urlencode "collection_concept_id=C1378227407-LAADS" \
     --data-urlencode "temporal=2017-01-01T00:00:00Z,2024-12-31T23:59:59Z" \
     --data-urlencode "bounding_box=-180,65,180,90" \
     --data-urlencode "page_size=500"  > result.json
```

1. 在 `result.json` 的每个 granule 里，`"links"` 数组有 HTTPS 下载地址（同样需要 Earthdata 认证 cookie 或 token）。
2. 翻页：增加 `page_num=2,3,…`，或者用 token-based “CMR-Search After” 机制。

------

## 3  下载后怎样投影成训练用影像？（简要提示）

1. 同步拉取 **MOD03/MYD03** geolocation 文件（与 02 同一天同 HHMM）。

2. Python 典型流程

   ```python
   from pyhdf.SD import SD         # 读 radiance band
   import h5py                     # 读 03 里的 Latitude/Longitude
   import rasterio, pyproj         # 投影、写 GeoTIFF
   ```

3. 投影极区常用 **极立体投影**：EPSG 3413（北极）/ 3031（南极）。

4. 每个 granule 约 2030×1354 px，先裁剪再拼接，最后写成 Cloud-Optimized GeoTIFF，便于深度学习框架按需读取。

------

## 4  示例：完整 Python 脚本（LAADS API 版）

> **用法**：
>
> ```bash
> python modis_polar_downloader.py arctic 2017 2024 /path/to/save
> # 或
> python modis_polar_downloader.py antarctic 2020 2021 ./polar_data
> ```

```python
#!/usr/bin/env python3
"""
批量下载极区 MODIS L1B (MOD021KM / MYD021KM) granule 的简单脚本
依赖: requests, tqdm
"""

import os, sys, json, time, argparse, datetime as dt
import requests
from tqdm import tqdm

LAADS_API = "https://ladsweb.modaps.eosdis.nasa.gov/api/v2"
PRODUCTS   = ["MOD021KM", "MYD021KM"]   # 可换成 HKM/QKM
COLL       = "61"

REGION_BBOX = {
    "arctic"   : "[BBOX]N90 S65 W-180 E180",
    "antarctic": "[BBOX]N-65 S-90 W-180 E180"
}

def query_granules(token:str, region:str, date_start:str, date_end:str):
    """
    调用 LAADS /content/details，返回 granule 元数据列表
    """
    url = f"{LAADS_API}/content/details"
    params = {
        "products"      : ",".join(PRODUCTS),
        "collections"   : COLL,
        "temporalRanges": f"{date_start}..{date_end}",
        "regions"       : REGION_BBOX[region],
        "formats"       : "json",
        "pageSize"      : "2000"      # 最大 2000
    }
    headers = {"Authorization": f"Bearer {token}"}
    r = requests.get(url, params=params, headers=headers, timeout=120)
    r.raise_for_status()
    return r.json()["files"]

def download_file(token:str, file_path:str, out_dir:str):
    """
    根据 archive 路径下载单个文件
    """
    url = f"{LAADS_API}/content/archives/{file_path}"
    local = os.path.join(out_dir, os.path.basename(file_path))
    if os.path.exists(local): return
    headers = {
        "Authorization": f"Bearer {token}",
        "X-Requested-With": "XMLHttpRequest"
    }
    with requests.get(url, headers=headers, stream=True, timeout=120) as r:
        r.raise_for_status()
        total = int(r.headers.get('content-length', 0))
        with open(local, 'wb') as f, tqdm(total=total, unit='B', unit_scale=True,
                                          desc=os.path.basename(local)) as pbar:
            for chunk in r.iter_content(chunk_size=8192):
                if chunk:
                    f.write(chunk)
                    pbar.update(len(chunk))

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("region", choices=["arctic", "antarctic"])
    parser.add_argument("year_start", type=int)
    parser.add_argument("year_end",   type=int)
    parser.add_argument("out_dir")
    args = parser.parse_args()

    token = os.getenv("LAADS_TOKEN")
    if not token:
        sys.exit("请先把 LAADS_TOKEN=你的token 写进环境变量！")

    os.makedirs(args.out_dir, exist_ok=True)

    date_start = f"{args.year_start}-01-01"
    date_end   = f"{args.year_end}-12-31"

    print("查询文件列表……")
    files = query_granules(token, args.region, date_start, date_end)
    print(f"共找到 {len(files)} 个 granule，开始下载……")

    for fmeta in files:
        download_file(token, fmeta["name"], args.out_dir)
        # 根据需要，这里也可顺带下载对应 MOD03/MYD03

if __name__ == "__main__":
    main()
```

> **脚本说明**
>
> 1. 登录 LAADS 生成的 **token** 放到环境变量：
>     `export LAADS_TOKEN=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`
> 2. `query_granules()` 调用 `/content/details`，过滤完只拿 `["files"]` 字段。
> 3. 返回的 `file_path` 已包含 “archive/allData/61/…” 相对路径，`download_file()` 用 `/content/archives/{file_path}` 直接下载。
> 4. `pageSize=2000` 如果单次返回超过 2000 条，可在脚本里再做翻页 (`&page=2`)。
> 5. **极区选择**：`arctic` / `antarctic` 决定 `regions=[BBOX]...`。
> 6. 如需 **MOD03/MYD03**，可以在 `download_file()` 里把文件名改成 `"MOD03" + 剩余部分` 再拉一份。

------

### 结束语

- **规律**：MOD02/MYD02 的“经纬度”不在文件名里，必须依赖官方 API 通过 **空间约束参数**（BBOX 或 bounding_box）先查元数据，再批量下载。
- 若要离线拼接、投影成图像，别忘了同步下载 geolocation 产品（03），用 Python/GDAL 投影到极立体坐标。
- 上面的脚本仅 100 行左右，可作为入门练习，后续可加入多线程、断点续传、自动拼接等功能。

祝你顺利构建训练集！如果还有任何细节想深入，随时再问。