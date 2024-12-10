# OpenPlatformService

## Flowchart

```mermaid
graph TD
  WebUI --Call API--> OpenPlatformService
  OpenPlatformService --Cron Fetch data--> EAS
  OpenPlatformService --Cron Sync data--> DB[(PostgreSQL)]
  OpenPlatformService --Query data--> DB[(PostgreSQL)]

```

## API

```
Req:
GET /v1/claimable-token?wallet=xxx

Resp:
[
  {
    "contractAddress": "xxx1", // projectId == contractAddress ?
    "wallet": "0x123"
    "token": "", // usdt, eth
    "amount": "10.23",
    "status": "",
    "ratio": 20, // 0-100
  },
  {
    "contractAddress": "xxx2", // projectId == contractAddress ?
    "wallet": "0x123"
    "token": "", // usdt, eth
    "amount": "10.23",
    "status": "",
    "ratio": 10, // 0-100
  }
]
```

## FAQ

- 目前主要获取每个 wallet 应该要 claim token 个数 ？只针对 incentive pool 情况么？
- EAS 需要同步的数据主要是有哪些？合约模板？用户数据？https://github.com/lxdao-official/fairsharing-contract 根据 schema 来么
- 同步时间是多久？1 天一次？12 个小时一次
- EAS 跑在 Optimism 链？测试在 Optimism-Sepolia？
- EAS all onchain 还是 部分 offchain ？我们需要只需要同步所有 onchain data 对吧？
- 读取 EAS，就可以使用 GraphQL API 根据 schema id 获取所有的关联的 tx id 和 Attestation UID？没有所谓的实时监听的需求
- 需要设置 DB ReadOnly Account

```
model IncentivePool {
  id           String                @id
  allocationId String
  projectId    String
  address      String
  creator      String
  timeToClaim  Int
  depositor    String
  wallets      String[]
  details      IncentivePoolDetail[]
  createAt     DateTime              @default(now())
  updatedAt    DateTime              @updatedAt
  deleted      Boolean               @default(false)
}

enum IncentiveCLAIMStatus {
  UNCLAIMED
  CLAIMED
}

model IncentivePoolDetail {
  id        String               @id @default(uuid())
  projectId    String
  poolId    String
  pool      IncentivePool        @relation(fields: [poolId], references: [id])
  token     String
  wallet    String
  amount    String
  ratio     Int
  status    IncentiveCLAIMStatus @default(UNCLAIMED)
  createAt  DateTime             @default(now())
  updatedAt DateTime             @updatedAt
  deleted   Boolean              @default(false)
}

```
