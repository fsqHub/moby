# Docker v1.13.1 及之后正式版本发布时间整理

数据来源：`https://github.com/moby/moby.git`。

整理规则：

- 起点包含 `v1.13.1` 本身。
- “正式版本”仅保留 Docker Engine 稳定版本标签：`vX.Y.Z`、`vX.Y.Z-ce`、`docker-vX.Y.Z`。
- 排除 `rc`、`beta`、`alpha`、`tp` 等预发布标签，排除 `client/*`、`api/*`、`docs/*` 等非 Docker Engine 版本标签。
- 发布时间优先取 GitHub Release 的 `published_at`；没有 GitHub Release 记录的历史稳定标签，取 Git tag 的 `creatordate`，并在来源列标注。
- 时间统一转换为 UTC，表格按版本号升序排列。

统计：共 190 个正式版本，其中 149 个来自 GitHub Release 发布时间，41 个来自 Git tag 时间。

## 大版本目录

- [1.x](#docker-1)：1 个版本，版本范围 `1.13.1` 至 `1.13.1`，时间范围 `2017-02-08T19:46:27Z` 至 `2017-02-08T19:46:27Z`。
- [17.x](#docker-17)：15 个版本，版本范围 `17.03.0-ce` 至 `17.12.1-ce`，时间范围 `2017-03-02T08:08:51Z` 至 `2018-02-07T23:25:50Z`。
- [18.x](#docker-18)：20 个版本，版本范围 `18.01.0-ce` 至 `18.09.9`，时间范围 `2017-12-12T05:41:05Z` 至 `2019-09-04T18:44:08Z`。
- [19.x](#docker-19)：16 个版本，版本范围 `19.03.0` 至 `19.03.15`，时间范围 `2019-07-22T18:25:55Z` 至 `2021-02-02T12:09:05Z`。
- [20.x](#docker-20)：28 个版本，版本范围 `20.10.0` 至 `20.10.27`，时间范围 `2020-12-09T22:26:06Z` 至 `2023-12-01T17:19:22Z`。
- [23.x](#docker-23)：19 个版本，版本范围 `23.0.0` 至 `23.0.18`，时间范围 `2023-02-02T11:20:15Z` 至 `2025-05-15T21:36:18Z`。
- [24.x](#docker-24)：10 个版本，版本范围 `24.0.0` 至 `24.0.9`，时间范围 `2023-05-16T17:36:10Z` 至 `2024-02-01T13:58:46Z`。
- [25.x](#docker-25)：17 个版本，版本范围 `25.0.0` 至 `25.0.16`，时间范围 `2024-01-19T12:58:49Z` 至 `2026-05-13T16:27:26Z`。
- [26.x](#docker-26)：9 个版本，版本范围 `26.0.0` 至 `26.1.5`，时间范围 `2024-03-20T19:03:46Z` 至 `2024-07-24T16:23:02Z`。
- [27.x](#docker-27)：14 个版本，版本范围 `27.0.1` 至 `27.5.1`，时间范围 `2024-06-25T09:59:03Z` 至 `2025-01-22T18:07:19Z`。
- [28.x](#docker-28)：18 个版本，版本范围 `28.0.0` 至 `28.5.2`，时间范围 `2025-02-20T01:23:15Z` 至 `2025-11-05T15:49:41Z`。
- [29.x](#docker-29)：23 个版本，版本范围 `29.0.0` 至 `29.5.3`，时间范围 `2025-11-10T22:45:57Z` 至 `2026-06-03T18:50:17Z`。
## 大版本统计

| 大版本 | 数量 | 起始版本 | 最新版本 | 时间范围（UTC） |
| --- | ---: | --- | --- | --- |
| [1.x](#docker-1) | 1 | `1.13.1` | `1.13.1` | `2017-02-08T19:46:27Z` 至 `2017-02-08T19:46:27Z` |
| [17.x](#docker-17) | 15 | `17.03.0-ce` | `17.12.1-ce` | `2017-03-02T08:08:51Z` 至 `2018-02-07T23:25:50Z` |
| [18.x](#docker-18) | 20 | `18.01.0-ce` | `18.09.9` | `2017-12-12T05:41:05Z` 至 `2019-09-04T18:44:08Z` |
| [19.x](#docker-19) | 16 | `19.03.0` | `19.03.15` | `2019-07-22T18:25:55Z` 至 `2021-02-02T12:09:05Z` |
| [20.x](#docker-20) | 28 | `20.10.0` | `20.10.27` | `2020-12-09T22:26:06Z` 至 `2023-12-01T17:19:22Z` |
| [23.x](#docker-23) | 19 | `23.0.0` | `23.0.18` | `2023-02-02T11:20:15Z` 至 `2025-05-15T21:36:18Z` |
| [24.x](#docker-24) | 10 | `24.0.0` | `24.0.9` | `2023-05-16T17:36:10Z` 至 `2024-02-01T13:58:46Z` |
| [25.x](#docker-25) | 17 | `25.0.0` | `25.0.16` | `2024-01-19T12:58:49Z` 至 `2026-05-13T16:27:26Z` |
| [26.x](#docker-26) | 9 | `26.0.0` | `26.1.5` | `2024-03-20T19:03:46Z` 至 `2024-07-24T16:23:02Z` |
| [27.x](#docker-27) | 14 | `27.0.1` | `27.5.1` | `2024-06-25T09:59:03Z` 至 `2025-01-22T18:07:19Z` |
| [28.x](#docker-28) | 18 | `28.0.0` | `28.5.2` | `2025-02-20T01:23:15Z` 至 `2025-11-05T15:49:41Z` |
| [29.x](#docker-29) | 23 | `29.0.0` | `29.5.3` | `2025-11-10T22:45:57Z` 至 `2026-06-03T18:50:17Z` |
<a id="docker-1"></a>
## 1.x

版本范围：`1.13.1` 至 `1.13.1`，共 1 个正式版本。

| 序号 | Docker 版本 | Git tag | 发布时间（UTC） | 时间来源 |
| ---: | --- | --- | --- | --- |
| 1 | `1.13.1` | `v1.13.1` | `2017-02-08T19:46:27Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v1.13.1) |

<a id="docker-17"></a>
## 17.x

版本范围：`17.03.0-ce` 至 `17.12.1-ce`，共 15 个正式版本。

| 序号 | Docker 版本 | Git tag | 发布时间（UTC） | 时间来源 |
| ---: | --- | --- | --- | --- |
| 2 | `17.03.0-ce` | `v17.03.0-ce` | `2017-03-02T08:08:51Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v17.03.0-ce) |
| 3 | `17.03.1-ce` | `v17.03.1-ce` | `2017-03-28T04:57:37Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v17.03.1-ce) |
| 4 | `17.03.2-ce` | `v17.03.2-ce` | `2017-06-28T03:57:02Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v17.03.2-ce) |
| 5 | `17.04.0-ce` | `v17.04.0-ce` | `2017-04-06T00:41:05Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v17.04.0-ce) |
| 6 | `17.05.0-ce` | `v17.05.0-ce` | `2017-05-05T18:09:56Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v17.05.0-ce) |
| 7 | `17.06.0-ce` | `v17.06.0-ce` | `2017-06-20T21:29:08Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v17.06.0-ce) |
| 8 | `17.06.1-ce` | `v17.06.1-ce` | `2017-08-17T17:56:19Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v17.06.1-ce) |
| 9 | `17.06.2-ce` | `v17.06.2-ce` | `2017-09-05T16:31:09Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v17.06.2-ce) |
| 10 | `17.07.0-ce` | `v17.07.0-ce` | `2017-08-28T22:58:55Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v17.07.0-ce) |
| 11 | `17.09.0-ce` | `v17.09.0-ce` | `2017-09-22T16:13:41Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v17.09.0-ce) |
| 12 | `17.09.1-ce` | `v17.09.1-ce` | `2017-12-07T22:04:54Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v17.09.1-ce) |
| 13 | `17.10.0-ce` | `v17.10.0-ce` | `2017-10-13T21:37:20Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v17.10.0-ce) |
| 14 | `17.11.0-ce` | `v17.11.0-ce` | `2017-11-17T21:37:55Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v17.11.0-ce) |
| 15 | `17.12.0-ce` | `v17.12.0-ce` | `2017-12-15T22:48:31Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v17.12.0-ce) |
| 16 | `17.12.1-ce` | `v17.12.1-ce` | `2018-02-07T23:25:50Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v17.12.1-ce) |

<a id="docker-18"></a>
## 18.x

版本范围：`18.01.0-ce` 至 `18.09.9`，共 20 个正式版本。

| 序号 | Docker 版本 | Git tag | 发布时间（UTC） | 时间来源 |
| ---: | --- | --- | --- | --- |
| 17 | `18.01.0-ce` | `v18.01.0-ce` | `2017-12-12T05:41:05Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v18.01.0-ce) |
| 18 | `18.02.0-ce` | `v18.02.0-ce` | `2018-01-26T21:15:36Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v18.02.0-ce) |
| 19 | `18.03.0-ce` | `v18.03.0-ce` | `2018-03-14T22:45:58Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v18.03.0-ce) |
| 20 | `18.03.1-ce` | `v18.03.1-ce` | `2018-04-25T22:09:31Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v18.03.1-ce) |
| 21 | `18.04.0-ce` | `v18.04.0-ce` | `2018-03-27T18:03:23Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v18.04.0-ce) |
| 22 | `18.05.0-ce` | `v18.05.0-ce` | `2018-04-25T21:30:40Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v18.05.0-ce) |
| 23 | `18.06.0-ce` | `v18.06.0-ce` | `2018-07-18T23:11:45Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v18.06.0-ce) |
| 24 | `18.06.1-ce` | `v18.06.1-ce` | `2018-08-21T23:34:42Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v18.06.1-ce) |
| 25 | `18.06.2-ce` | `v18.06.2-ce` | `2019-02-11T18:44:19Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v18.06.2-ce) |
| 26 | `18.06.3-ce` | `v18.06.3-ce` | `2019-02-20T18:07:02Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v18.06.3-ce) |
| 27 | `18.09.0` | `v18.09.0` | `2018-11-08T00:24:59Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v18.09.0) |
| 28 | `18.09.1` | `v18.09.1` | `2019-01-09T21:33:50Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v18.09.1) |
| 29 | `18.09.2` | `v18.09.2` | `2019-02-11T16:59:55Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v18.09.2) |
| 30 | `18.09.3` | `v18.09.3` | `2019-02-28T18:05:41Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v18.09.3) |
| 31 | `18.09.4` | `v18.09.4` | `2019-03-28T05:18:58Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v18.09.4) |
| 32 | `18.09.5` | `v18.09.5` | `2019-04-11T07:07:08Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v18.09.5) |
| 33 | `18.09.6` | `v18.09.6` | `2019-05-06T17:21:47Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v18.09.6) |
| 34 | `18.09.7` | `v18.09.7` | `2019-06-27T19:50:48Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v18.09.7) |
| 35 | `18.09.8` | `v18.09.8` | `2019-07-17T19:13:43Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v18.09.8) |
| 36 | `18.09.9` | `v18.09.9` | `2019-09-04T18:44:08Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v18.09.9) |

<a id="docker-19"></a>
## 19.x

版本范围：`19.03.0` 至 `19.03.15`，共 16 个正式版本。

| 序号 | Docker 版本 | Git tag | 发布时间（UTC） | 时间来源 |
| ---: | --- | --- | --- | --- |
| 37 | `19.03.0` | `v19.03.0` | `2019-07-22T18:25:55Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v19.03.0) |
| 38 | `19.03.1` | `v19.03.1` | `2019-07-26T01:16:41Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v19.03.1) |
| 39 | `19.03.2` | `v19.03.2` | `2019-09-03T19:54:20Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v19.03.2) |
| 40 | `19.03.3` | `v19.03.3` | `2019-10-08T17:54:26Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v19.03.3) |
| 41 | `19.03.4` | `v19.03.4` | `2019-10-18T17:45:33Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v19.03.4) |
| 42 | `19.03.5` | `v19.03.5` | `2019-11-14T18:41:12Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v19.03.5) |
| 43 | `19.03.6` | `v19.03.6` | `2020-02-13T03:19:47Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v19.03.6) |
| 44 | `19.03.7` | `v19.03.7` | `2020-03-04T05:56:15Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v19.03.7) |
| 45 | `19.03.8` | `v19.03.8` | `2020-04-09T18:28:28Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v19.03.8) |
| 46 | `19.03.9` | `v19.03.9` | `2020-05-28T18:10:58Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v19.03.9) |
| 47 | `19.03.10` | `v19.03.10` | `2020-05-29T18:22:55Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v19.03.10) |
| 48 | `19.03.11` | `v19.03.11` | `2020-06-04T19:58:57Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v19.03.11) |
| 49 | `19.03.12` | `v19.03.12` | `2020-06-30T12:15:06Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v19.03.12) |
| 50 | `19.03.13` | `v19.03.13` | `2020-09-17T19:58:57Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v19.03.13) |
| 51 | `19.03.14` | `v19.03.14` | `2020-12-02T08:24:53Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v19.03.14) |
| 52 | `19.03.15` | `v19.03.15` | `2021-02-02T12:09:05Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v19.03.15) |

<a id="docker-20"></a>
## 20.x

版本范围：`20.10.0` 至 `20.10.27`，共 28 个正式版本。

| 序号 | Docker 版本 | Git tag | 发布时间（UTC） | 时间来源 |
| ---: | --- | --- | --- | --- |
| 53 | `20.10.0` | `v20.10.0` | `2020-12-09T22:26:06Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v20.10.0) |
| 54 | `20.10.1` | `v20.10.1` | `2020-12-15T10:07:17Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v20.10.1) |
| 55 | `20.10.2` | `v20.10.2` | `2021-01-05T10:55:26Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v20.10.2) |
| 56 | `20.10.3` | `v20.10.3` | `2021-02-02T12:09:35Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v20.10.3) |
| 57 | `20.10.4` | `v20.10.4` | `2021-02-28T14:23:08Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v20.10.4) |
| 58 | `20.10.5` | `v20.10.5` | `2021-03-03T22:19:33Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v20.10.5) |
| 59 | `20.10.6` | `v20.10.6` | `2021-04-14T19:57:30Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v20.10.6) |
| 60 | `20.10.7` | `v20.10.7` | `2021-06-02T20:42:35Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v20.10.7) |
| 61 | `20.10.8` | `v20.10.8` | `2021-08-04T00:23:18Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v20.10.8) |
| 62 | `20.10.9` | `v20.10.9` | `2021-10-04T18:31:16Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v20.10.9) |
| 63 | `20.10.10` | `v20.10.10` | `2021-10-25T16:15:34Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v20.10.10) |
| 64 | `20.10.11` | `v20.10.11` | `2021-11-18T02:15:07Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v20.10.11) |
| 65 | `20.10.12` | `v20.10.12` | `2022-01-10T10:09:34Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v20.10.12) |
| 66 | `20.10.13` | `v20.10.13` | `2022-03-10T18:59:43Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v20.10.13) |
| 67 | `20.10.14` | `v20.10.14` | `2022-03-24T03:29:00Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v20.10.14) |
| 68 | `20.10.15` | `v20.10.15` | `2022-05-05T22:01:09Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v20.10.15) |
| 69 | `20.10.16` | `v20.10.16` | `2022-05-12T15:41:58Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v20.10.16) |
| 70 | `20.10.17` | `v20.10.17` | `2022-06-07T01:30:04Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v20.10.17) |
| 71 | `20.10.18` | `v20.10.18` | `2022-09-09T09:45:55Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v20.10.18) |
| 72 | `20.10.19` | `v20.10.19` | `2022-10-13T23:22:12Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v20.10.19) |
| 73 | `20.10.20` | `v20.10.20` | `2022-10-18T21:19:39Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v20.10.20) |
| 74 | `20.10.21` | `v20.10.21` | `2022-10-25T21:44:27Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v20.10.21) |
| 75 | `20.10.22` | `v20.10.22` | `2022-12-16T14:44:15Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v20.10.22) |
| 76 | `20.10.23` | `v20.10.23` | `2023-01-20T00:15:25Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v20.10.23) |
| 77 | `20.10.24` | `v20.10.24` | `2023-04-04T21:04:57Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v20.10.24) |
| 78 | `20.10.25` | `v20.10.25` | `2023-05-15T22:45:15Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v20.10.25) |
| 79 | `20.10.26` | `v20.10.26` | `2023-09-27T21:21:03Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v20.10.26) |
| 80 | `20.10.27` | `v20.10.27` | `2023-12-01T17:19:22Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v20.10.27) |

<a id="docker-23"></a>
## 23.x

版本范围：`23.0.0` 至 `23.0.18`，共 19 个正式版本。

| 序号 | Docker 版本 | Git tag | 发布时间（UTC） | 时间来源 |
| ---: | --- | --- | --- | --- |
| 81 | `23.0.0` | `v23.0.0` | `2023-02-02T11:20:15Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v23.0.0) |
| 82 | `23.0.1` | `v23.0.1` | `2023-02-10T00:15:52Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v23.0.1) |
| 83 | `23.0.2` | `v23.0.2` | `2023-03-28T12:12:48Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v23.0.2) |
| 84 | `23.0.3` | `v23.0.3` | `2023-04-05T00:25:10Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v23.0.3) |
| 85 | `23.0.4` | `v23.0.4` | `2023-04-17T19:56:03Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v23.0.4) |
| 86 | `23.0.5` | `v23.0.5` | `2023-04-26T19:57:15Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v23.0.5) |
| 87 | `23.0.6` | `v23.0.6` | `2023-05-08T11:34:04Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v23.0.6) |
| 88 | `23.0.7` | `v23.0.7` | `2023-09-27T21:21:24Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v23.0.7) |
| 89 | `23.0.8` | `v23.0.8` | `2023-12-01T17:12:07Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v23.0.8) |
| 90 | `23.0.9` | `v23.0.9` | `2024-01-31T00:19:26Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v23.0.9) |
| 91 | `23.0.10` | `v23.0.10` | `2024-03-21T17:04:17Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v23.0.10) |
| 92 | `23.0.11` | `v23.0.11` | `2024-05-06T15:08:45Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v23.0.11) |
| 93 | `23.0.12` | `v23.0.12` | `2024-05-29T17:27:53Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v23.0.12) |
| 94 | `23.0.13` | `v23.0.13` | `2024-06-20T18:03:00Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v23.0.13) |
| 95 | `23.0.14` | `v23.0.14` | `2024-08-19T18:21:14Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v23.0.14) |
| 96 | `23.0.15` | `v23.0.15` | `2024-10-07T16:23:54Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v23.0.15) |
| 97 | `23.0.16` | `v23.0.16` | `2024-12-05T19:33:16Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v23.0.16) |
| 98 | `23.0.17` | `v23.0.17` | `2025-05-15T21:34:56Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v23.0.17) |
| 99 | `23.0.18` | `v23.0.18` | `2025-05-15T21:36:18Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v23.0.18) |

<a id="docker-24"></a>
## 24.x

版本范围：`24.0.0` 至 `24.0.9`，共 10 个正式版本。

| 序号 | Docker 版本 | Git tag | 发布时间（UTC） | 时间来源 |
| ---: | --- | --- | --- | --- |
| 100 | `24.0.0` | `v24.0.0` | `2023-05-16T17:36:10Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v24.0.0) |
| 101 | `24.0.1` | `v24.0.1` | `2023-05-19T23:38:18Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v24.0.1) |
| 102 | `24.0.2` | `v24.0.2` | `2023-05-26T08:55:54Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v24.0.2) |
| 103 | `24.0.3` | `v24.0.3` | `2023-07-06T17:37:31Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v24.0.3) |
| 104 | `24.0.4` | `v24.0.4` | `2023-07-07T21:13:35Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v24.0.4) |
| 105 | `24.0.5` | `v24.0.5` | `2023-07-24T16:05:10Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v24.0.5) |
| 106 | `24.0.6` | `v24.0.6` | `2023-09-05T21:35:42Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v24.0.6) |
| 107 | `24.0.7` | `v24.0.7` | `2023-10-27T11:45:59Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v24.0.7) |
| 108 | `24.0.8` | `v24.0.8` | `2024-01-25T22:52:45Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v24.0.8) |
| 109 | `24.0.9` | `v24.0.9` | `2024-02-01T13:58:46Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v24.0.9) |

<a id="docker-25"></a>
## 25.x

版本范围：`25.0.0` 至 `25.0.16`，共 17 个正式版本。

| 序号 | Docker 版本 | Git tag | 发布时间（UTC） | 时间来源 |
| ---: | --- | --- | --- | --- |
| 110 | `25.0.0` | `v25.0.0` | `2024-01-19T12:58:49Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v25.0.0) |
| 111 | `25.0.1` | `v25.0.1` | `2024-01-24T10:28:36Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v25.0.1) |
| 112 | `25.0.2` | `v25.0.2` | `2024-02-01T02:46:02Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v25.0.2) |
| 113 | `25.0.3` | `v25.0.3` | `2024-02-07T00:41:55Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v25.0.3) |
| 114 | `25.0.4` | `v25.0.4` | `2024-03-07T11:01:32Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v25.0.4) |
| 115 | `25.0.5` | `v25.0.5` | `2024-03-19T21:36:29Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v25.0.5) |
| 116 | `25.0.6` | `v25.0.6` | `2024-07-25T19:43:11Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v25.0.6) |
| 117 | `25.0.7` | `v25.0.7` | `2024-12-05T19:38:17Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v25.0.7) |
| 118 | `25.0.8` | `v25.0.8` | `2025-02-03T05:43:39Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v25.0.8) |
| 119 | `25.0.9` | `v25.0.9` | `2025-05-15T21:18:47Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v25.0.9) |
| 120 | `25.0.10` | `v25.0.10` | `2025-05-15T21:23:43Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v25.0.10) |
| 121 | `25.0.11` | `v25.0.11` | `2025-06-18T23:19:19Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v25.0.11) |
| 122 | `25.0.12` | `v25.0.12` | `2025-07-15T18:32:42Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v25.0.12) |
| 123 | `25.0.13` | `v25.0.13` | `2025-09-04T14:34:06Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v25.0.13) |
| 124 | `25.0.14` | `v25.0.14` | `2025-11-06T23:52:23Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v25.0.14) |
| 125 | `25.0.15` | `v25.0.15` | `2026-04-28T14:47:33Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v25.0.15) |
| 126 | `25.0.16` | `v25.0.16` | `2026-05-13T16:27:26Z` | [Git tag creatordate](https://github.com/moby/moby/tree/v25.0.16) |

<a id="docker-26"></a>
## 26.x

版本范围：`26.0.0` 至 `26.1.5`，共 9 个正式版本。

| 序号 | Docker 版本 | Git tag | 发布时间（UTC） | 时间来源 |
| ---: | --- | --- | --- | --- |
| 127 | `26.0.0` | `v26.0.0` | `2024-03-20T19:03:46Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v26.0.0) |
| 128 | `26.0.1` | `v26.0.1` | `2024-04-11T14:54:26Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v26.0.1) |
| 129 | `26.0.2` | `v26.0.2` | `2024-04-18T20:35:25Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v26.0.2) |
| 130 | `26.1.0` | `v26.1.0` | `2024-04-22T21:11:27Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v26.1.0) |
| 131 | `26.1.1` | `v26.1.1` | `2024-04-30T16:03:27Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v26.1.1) |
| 132 | `26.1.2` | `v26.1.2` | `2024-05-09T10:53:46Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v26.1.2) |
| 133 | `26.1.3` | `v26.1.3` | `2024-05-16T13:28:41Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v26.1.3) |
| 134 | `26.1.4` | `v26.1.4` | `2024-06-05T18:31:24Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v26.1.4) |
| 135 | `26.1.5` | `v26.1.5` | `2024-07-24T16:23:02Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v26.1.5) |

<a id="docker-27"></a>
## 27.x

版本范围：`27.0.1` 至 `27.5.1`，共 14 个正式版本。

| 序号 | Docker 版本 | Git tag | 发布时间（UTC） | 时间来源 |
| ---: | --- | --- | --- | --- |
| 136 | `27.0.1` | `v27.0.1` | `2024-06-25T09:59:03Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v27.0.1) |
| 137 | `27.0.2` | `v27.0.2` | `2024-06-26T22:04:18Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v27.0.2) |
| 138 | `27.0.3` | `v27.0.3` | `2024-07-01T09:47:51Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v27.0.3) |
| 139 | `27.1.0` | `v27.1.0` | `2024-07-22T11:54:59Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v27.1.0) |
| 140 | `27.1.1` | `v27.1.1` | `2024-07-23T22:17:08Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v27.1.1) |
| 141 | `27.1.2` | `v27.1.2` | `2024-08-13T14:19:32Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v27.1.2) |
| 142 | `27.2.0` | `v27.2.0` | `2024-08-27T20:19:04Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v27.2.0) |
| 143 | `27.2.1` | `v27.2.1` | `2024-09-09T11:18:15Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v27.2.1) |
| 144 | `27.3.0` | `v27.3.0` | `2024-09-19T20:00:33Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v27.3.0) |
| 145 | `27.3.1` | `v27.3.1` | `2024-09-20T18:14:36Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v27.3.1) |
| 146 | `27.4.0` | `v27.4.0` | `2024-12-09T15:54:12Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v27.4.0) |
| 147 | `27.4.1` | `v27.4.1` | `2024-12-18T11:35:46Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v27.4.1) |
| 148 | `27.5.0` | `v27.5.0` | `2025-01-13T18:46:13Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v27.5.0) |
| 149 | `27.5.1` | `v27.5.1` | `2025-01-22T18:07:19Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v27.5.1) |

<a id="docker-28"></a>
## 28.x

版本范围：`28.0.0` 至 `28.5.2`，共 18 个正式版本。

| 序号 | Docker 版本 | Git tag | 发布时间（UTC） | 时间来源 |
| ---: | --- | --- | --- | --- |
| 150 | `28.0.0` | `v28.0.0` | `2025-02-20T01:23:15Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v28.0.0) |
| 151 | `28.0.1` | `v28.0.1` | `2025-02-26T14:22:34Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v28.0.1) |
| 152 | `28.0.2` | `v28.0.2` | `2025-03-19T16:26:37Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v28.0.2) |
| 153 | `28.0.3` | `v28.0.3` | `2025-03-25T13:26:01Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v28.0.3) |
| 154 | `28.0.4` | `v28.0.4` | `2025-03-25T16:54:19Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v28.0.4) |
| 155 | `28.1.0` | `v28.1.0` | `2025-04-17T13:35:20Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v28.1.0) |
| 156 | `28.1.1` | `v28.1.1` | `2025-04-18T11:42:54Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v28.1.1) |
| 157 | `28.2.0` | `v28.2.0` | `2025-05-28T17:47:06Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v28.2.0) |
| 158 | `28.2.1` | `v28.2.1` | `2025-05-28T22:23:22Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v28.2.1) |
| 159 | `28.2.2` | `v28.2.2` | `2025-05-30T15:14:55Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v28.2.2) |
| 160 | `28.3.0` | `v28.3.0` | `2025-06-25T00:17:12Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v28.3.0) |
| 161 | `28.3.1` | `v28.3.1` | `2025-07-02T23:33:55Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v28.3.1) |
| 162 | `28.3.2` | `v28.3.2` | `2025-07-09T19:47:46Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v28.3.2) |
| 163 | `28.3.3` | `v28.3.3` | `2025-07-29T09:52:24Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v28.3.3) |
| 164 | `28.4.0` | `v28.4.0` | `2025-09-03T21:52:07Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v28.4.0) |
| 165 | `28.5.0` | `v28.5.0` | `2025-10-02T20:12:31Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v28.5.0) |
| 166 | `28.5.1` | `v28.5.1` | `2025-10-08T13:09:36Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v28.5.1) |
| 167 | `28.5.2` | `v28.5.2` | `2025-11-05T15:49:41Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/v28.5.2) |

<a id="docker-29"></a>
## 29.x

版本范围：`29.0.0` 至 `29.5.3`，共 23 个正式版本。

| 序号 | Docker 版本 | Git tag | 发布时间（UTC） | 时间来源 |
| ---: | --- | --- | --- | --- |
| 168 | `29.0.0` | `docker-v29.0.0` | `2025-11-10T22:45:57Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/docker-v29.0.0) |
| 169 | `29.0.1` | `docker-v29.0.1` | `2025-11-14T16:57:56Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/docker-v29.0.1) |
| 170 | `29.0.2` | `docker-v29.0.2` | `2025-11-17T16:56:16Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/docker-v29.0.2) |
| 171 | `29.0.3` | `docker-v29.0.3` | `2025-11-24T11:45:32Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/docker-v29.0.3) |
| 172 | `29.0.4` | `docker-v29.0.4` | `2025-11-24T22:27:01Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/docker-v29.0.4) |
| 173 | `29.1.0` | `docker-v29.1.0` | `2025-11-27T17:35:17Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/docker-v29.1.0) |
| 174 | `29.1.1` | `docker-v29.1.1` | `2025-11-28T13:08:46Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/docker-v29.1.1) |
| 175 | `29.1.2` | `docker-v29.1.2` | `2025-12-02T22:36:58Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/docker-v29.1.2) |
| 176 | `29.1.3` | `docker-v29.1.3` | `2025-12-12T15:56:35Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/docker-v29.1.3) |
| 177 | `29.1.4` | `docker-v29.1.4` | `2026-01-09T09:33:14Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/docker-v29.1.4) |
| 178 | `29.1.5` | `docker-v29.1.5` | `2026-01-16T14:08:54Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/docker-v29.1.5) |
| 179 | `29.2.0` | `docker-v29.2.0` | `2026-01-26T21:09:53Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/docker-v29.2.0) |
| 180 | `29.2.1` | `docker-v29.2.1` | `2026-02-02T18:16:53Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/docker-v29.2.1) |
| 181 | `29.3.0` | `docker-v29.3.0` | `2026-03-05T15:27:21Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/docker-v29.3.0) |
| 182 | `29.3.1` | `docker-v29.3.1` | `2026-03-25T17:46:19Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/docker-v29.3.1) |
| 183 | `29.4.0` | `docker-v29.4.0` | `2026-04-07T09:22:47Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/docker-v29.4.0) |
| 184 | `29.4.1` | `docker-v29.4.1` | `2026-04-20T16:45:55Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/docker-v29.4.1) |
| 185 | `29.4.2` | `docker-v29.4.2` | `2026-05-01T07:40:30Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/docker-v29.4.2) |
| 186 | `29.4.3` | `docker-v29.4.3` | `2026-05-06T17:48:41Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/docker-v29.4.3) |
| 187 | `29.5.0` | `docker-v29.5.0` | `2026-05-14T21:37:30Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/docker-v29.5.0) |
| 188 | `29.5.1` | `docker-v29.5.1` | `2026-05-18T17:10:35Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/docker-v29.5.1) |
| 189 | `29.5.2` | `docker-v29.5.2` | `2026-05-20T18:02:30Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/docker-v29.5.2) |
| 190 | `29.5.3` | `docker-v29.5.3` | `2026-06-03T18:50:17Z` | [GitHub Release](https://github.com/moby/moby/releases/tag/docker-v29.5.3) |

备注：GitHub Release 记录在早期 17.x/18.x/19.03 前半段并不完整，因此不能只依赖 Releases API；缺失 release 页的稳定 tag 仍属于仓库中的正式版本标签。
