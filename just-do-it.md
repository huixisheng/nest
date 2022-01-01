# 阅读
> nestjs 源码阅读记录

## 步骤
1. fork 
2. projj add git@github.com:huixisheng/nest.git
3. 阅读 CONTRIBUTING.md
4. yarn install 
5. sh scripts/prepare.sh
6. git checkout just-do-it

### scripts/prepare.sh

代码构建使用 gulp。`tools/gulp/gulpfile`。注册 `ts-node` 使得 gulp 的 task 支持 ts 语法。包含 5 个 tasks:

- Copies assets like Readme.md or LICENSE from the project base path to all the packages. 拷贝 'Readme.md', 'LICENSE', '.npmignore' 到 packages 每一个包中。
- Cleans the build output assets from the packages folders
- packages 下的内容编译转换
  - Builds the given package
  - Builds the given package and adds sourcemaps
- Moves the compiled nest files into the `samples/*` dirs.
- samples 下执行 install、build、test、test:e2e 等命令

每个 tasks 会注册不同的 gulp 的 task，用于 package.json 的 scripts 命令执行。


用 docker 初始化 samples 需要的环境。

`"test:docker:up": "docker-compose -f integration/docker-compose.yml up -d",`

```bash
version: "3"

services:
  redis:
    container_name: test-redis
    image: redis
    ports:
      - "6379:6379"
    restart: always
  nats:
    container_name: test-nats
    image: nats
    ports:
      - "8222:8222"
      - "4222:4222"
      - "6222:6222"
    restart: always
  mqtt:
    container_name: test-mqtt
    image: toke/mosquitto
    ports:
      - "1883:1883"
      - "9001:9001"
    restart: always
  mysql:
    image: mysql:8.0.27
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: test
    ports:
      - "3306:3306"
    restart: always
  mongodb:
    container_name: test-mongodb
    image: mongo:latest
    environment:
      - MONGODB_DATABASE="test"
    ports:
      - 27017:27017
  rabbit:
    container_name: test-rabbit
    hostname: rabbit
    image: "rabbitmq:management"
    ports:
      - "15672:15672"
      - "5672:5672"
    tty: true
  zookeeper:
    container_name: test-zookeeper
    hostname: zookeeper
    image: confluentinc/cp-zookeeper:7.0.1
    ports:
      - "2181:2181"
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
  kafka:
    container_name: test-kafka
    hostname: kafka
    image: confluentinc/cp-kafka:7.0.1
    depends_on:
      - zookeeper
    ports:
      - "29092:29092"
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_DELETE_TOPIC_ENABLE: 'true'

```


## 学到啥

1. 关键函数有注释。
2. 适当的使用 shell。
3. gulp 和 ts-node 结合使用。
4. 公共的配置。`tools/gulp/config.ts`。
5. 完善的单元测试。
6. benchmarks
7. 完善的配套设施。.circleci、.github、benchmarks、integration、scripts、.prettierrc、eslint、.commitlintrc、lerna、mocha

npm run build 执行了 gulp build、gulp move。


## 使用的核心库
concurrently、nyc、mocha


## 目录结构