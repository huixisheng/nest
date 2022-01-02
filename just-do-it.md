# 阅读
> nestjs 源码阅读记录

## 为何做这个事情
> 工作项目中 nestjs 应用比较多，当文档重读一篇之后发现有好多的特性不清楚，同时对于文档里面实现的新特性非常好奇是如何实现的。为了知其然而知其所以然，同时学习著名框架的精髓，因此有了这个文件，通过 just-do-it.md 文档记录学习的过程。

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
6. benchmarks 基准测试。
7. 完善的配套设施。.circleci、.github、benchmarks、integration、scripts、.prettierrc、eslint、.commitlintrc、lerna、mocha、docker、docker-compose。
8. opencollective 
9. 设计模式: 适配器、工厂模式
10. @types 写在 devDependencies
11. 实时监测 typescript 转换，ts-node、nodemon

npm run build 执行了 gulp build、gulp move。


## 使用的核心库
concurrently、nyc、mocha、codechecks、lerna-changelog


## 📃 目录结构

`tree -L 3 -I 'node_modules|.git|package-lock.json' -a`

> brew install tree

```bash
.
├── .circleci
│   ├── config.yml
│   └── install-wrk.sh
├── .commitlintrc.json
├── .eslintignore
├── .eslintrc.js
├── .eslintrc.spec.js
├── .gitattributes
├── .github
│   ├── FUNDING.yml
│   ├── ISSUE_TEMPLATE
│   │   ├── Bug_report.yml
│   │   ├── Feature_request.yml
│   │   ├── Regression.yml
│   │   ├── Suggestion_improve_performance.yml
│   │   └── config.yml
│   ├── PULL_REQUEST_TEMPLATE.md
│   ├── dependabot.yml
│   ├── lock.yml
│   └── workflows
│       └── codeql-analysis.yml
├── .gitignore
├── .husky
│   ├── _
│   │   ├── .gitignore
│   │   └── husky.sh
│   ├── commit-msg
│   └── pre-commit
├── .mocharc.js
├── .npmignore
├── .prettierignore
├── .prettierrc
├── CODE_OF_CONDUCT.md
├── CONTRIBUTING.md
├── LICENSE
├── Readme.md
├── SECURITY.md
├── benchmarks
│   ├── all_output.txt
│   ├── express.js
│   ├── fastify.js
│   ├── nest
│   │   ├── app.controller.d.ts
│   │   ├── app.controller.js
│   │   ├── app.controller.js.map
│   │   ├── app.module.d.ts
│   │   ├── app.module.js
│   │   ├── app.module.js.map
│   │   ├── main.d.ts
│   │   └── main.js.map
│   ├── nest-fastify.js
│   └── nest.js
├── gulpfile.js
├── integration
│   ├── auto-mock
│   │   ├── src
│   │   ├── test
│   │   └── tsconfig.json
│   ├── cache
│   │   ├── e2e
│   │   └── src
│   ├── cors
│   │   ├── e2e
│   │   ├── src
│   │   └── tsconfig.json
│   ├── docker-compose.yml
│   ├── graphql-code-first
│   │   ├── e2e
│   │   ├── schema.gql
│   │   ├── src
│   │   └── tsconfig.json
│   ├── graphql-schema-first
│   │   ├── e2e
│   │   ├── src
│   │   └── tsconfig.json
│   ├── hello-world
│   │   ├── e2e
│   │   ├── src
│   │   └── tsconfig.json
│   ├── hooks
│   │   ├── e2e
│   │   ├── src
│   │   └── tsconfig.json
│   ├── injector
│   │   ├── e2e
│   │   ├── src
│   │   └── tsconfig.json
│   ├── microservices
│   │   ├── e2e
│   │   ├── src
│   │   └── tsconfig.json
│   ├── mongoose
│   │   ├── e2e
│   │   ├── src
│   │   └── tsconfig.json
│   ├── nest-application
│   │   ├── app-locals
│   │   ├── get-url
│   │   ├── global-prefix
│   │   └── listen
│   ├── scopes
│   │   ├── e2e
│   │   ├── src
│   │   └── tsconfig.json
│   ├── send-files
│   │   ├── e2e
│   │   ├── src
│   │   └── tsconfig.json
│   ├── typeorm
│   │   ├── e2e
│   │   ├── ormconfig.json
│   │   ├── src
│   │   └── tsconfig.json
│   ├── versioning
│   │   ├── e2e
│   │   ├── src
│   │   └── tsconfig.json
│   └── websockets
│       ├── e2e
│       ├── src
│       └── tsconfig.json
├── just-do-it.md
├── lerna.json
├── package.json
├── packages
│   ├── common
│   │   ├── CHANGELOG.md
│   │   ├── PACKAGE.md
│   │   ├── Readme.md
│   │   ├── cache
│   │   ├── constants.ts
│   │   ├── decorators
│   │   ├── enums
│   │   ├── exceptions
│   │   ├── file-stream
│   │   ├── http
│   │   ├── index.ts
│   │   ├── interfaces
│   │   ├── package.json
│   │   ├── pipes
│   │   ├── serializer
│   │   ├── services
│   │   ├── test
│   │   ├── tsconfig.json
│   │   └── utils
│   ├── core
│   │   ├── CHANGELOG.md
│   │   ├── PACKAGE.md
│   │   ├── Readme.md
│   │   ├── adapters
│   │   ├── application-config.ts
│   │   ├── constants.ts
│   │   ├── discovery
│   │   ├── errors
│   │   ├── exceptions
│   │   ├── guards
│   │   ├── helpers
│   │   ├── hooks
│   │   ├── index.ts
│   │   ├── injector
│   │   ├── interceptors
│   │   ├── metadata-scanner.ts
│   │   ├── middleware
│   │   ├── nest-application-context.ts
│   │   ├── nest-application.ts
│   │   ├── nest-factory.ts
│   │   ├── package.json
│   │   ├── pipes
│   │   ├── router
│   │   ├── scanner.ts
│   │   ├── services
│   │   ├── test
│   │   └── tsconfig.json
│   ├── index.ts
│   ├── microservices
│   │   ├── CHANGELOG.md
│   │   ├── Readme.md
│   │   ├── client
│   │   ├── constants.ts
│   │   ├── container.ts
│   │   ├── context
│   │   ├── ctx-host
│   │   ├── decorators
│   │   ├── deserializers
│   │   ├── enums
│   │   ├── errors
│   │   ├── exceptions
│   │   ├── external
│   │   ├── factories
│   │   ├── helpers
│   │   ├── index.ts
│   │   ├── interfaces
│   │   ├── listener-metadata-explorer.ts
│   │   ├── listeners-controller.ts
│   │   ├── microservices-module.ts
│   │   ├── module
│   │   ├── nest-microservice.ts
│   │   ├── package.json
│   │   ├── record-builders
│   │   ├── serializers
│   │   ├── server
│   │   ├── test
│   │   ├── tokens.ts
│   │   ├── tsconfig.json
│   │   └── utils
│   ├── platform-express
│   │   ├── CHANGELOG.md
│   │   ├── Readme.md
│   │   ├── adapters
│   │   ├── index.ts
│   │   ├── interfaces
│   │   ├── multer
│   │   ├── package.json
│   │   ├── test
│   │   └── tsconfig.json
│   ├── platform-fastify
│   │   ├── CHANGELOG.md
│   │   ├── Readme.md
│   │   ├── adapters
│   │   ├── index.ts
│   │   ├── interfaces
│   │   ├── package.json
│   │   └── tsconfig.json
│   ├── platform-socket.io
│   │   ├── CHANGELOG.md
│   │   ├── Readme.md
│   │   ├── adapters
│   │   ├── index.ts
│   │   ├── package.json
│   │   └── tsconfig.json
│   ├── platform-ws
│   │   ├── CHANGELOG.md
│   │   ├── Readme.md
│   │   ├── adapters
│   │   ├── index.ts
│   │   ├── package.json
│   │   └── tsconfig.json
│   ├── testing
│   │   ├── CHANGELOG.md
│   │   ├── Readme.md
│   │   ├── index.ts
│   │   ├── interfaces
│   │   ├── package.json
│   │   ├── services
│   │   ├── test.ts
│   │   ├── testing-injector.ts
│   │   ├── testing-instance-loader.ts
│   │   ├── testing-module.builder.ts
│   │   ├── testing-module.ts
│   │   └── tsconfig.json
│   ├── tsconfig.base.json
│   └── websockets
│       ├── CHANGELOG.md
│       ├── Readme.md
│       ├── adapters
│       ├── constants.ts
│       ├── context
│       ├── decorators
│       ├── enums
│       ├── errors
│       ├── exceptions
│       ├── factories
│       ├── gateway-metadata-explorer.ts
│       ├── index.ts
│       ├── interfaces
│       ├── package.json
│       ├── socket-module.ts
│       ├── socket-server-provider.ts
│       ├── sockets-container.ts
│       ├── test
│       ├── tsconfig.json
│       ├── utils
│       └── web-sockets-controller.ts
├── readme_jp.md
├── readme_kr.md
├── readme_zh.md
├── renovate.json
├── sample
│   ├── 01-cats-app
│   │   ├── .eslintrc.js
│   │   ├── .gitignore
│   │   ├── e2e
│   │   ├── jest.json
│   │   ├── package.json
│   │   ├── src
│   │   ├── tsconfig.build.json
│   │   └── tsconfig.json
│   ├── 02-gateways
│   │   ├── .eslintrc.js
│   │   ├── .gitignore
│   │   ├── client
│   │   ├── package.json
│   │   ├── src
│   │   ├── tsconfig.build.json
│   │   └── tsconfig.json
│   ├── 03-microservices
│   │   ├── .eslintrc.js
│   │   ├── .gitignore
│   │   ├── package.json
│   │   ├── src
│   │   ├── tsconfig.build.json
│   │   └── tsconfig.json
│   ├── 04-grpc
│   │   ├── .eslintrc.js
│   │   ├── .gitignore
│   │   ├── nest-cli.json
│   │   ├── package.json
│   │   ├── src
│   │   ├── tsconfig.build.json
│   │   └── tsconfig.json
│   ├── 05-sql-typeorm
│   │   ├── .eslintrc.js
│   │   ├── .gitignore
│   │   ├── README.md
│   │   ├── docker-compose.yml
│   │   ├── jest.json
│   │   ├── package.json
│   │   ├── src
│   │   ├── tsconfig.build.json
│   │   └── tsconfig.json
│   ├── 06-mongoose
│   │   ├── .eslintrc.js
│   │   ├── .gitignore
│   │   ├── README.md
│   │   ├── docker-compose.yml
│   │   ├── package.json
│   │   ├── src
│   │   ├── tsconfig.build.json
│   │   └── tsconfig.json
│   ├── 07-sequelize
│   │   ├── .eslintrc.js
│   │   ├── .gitignore
│   │   ├── README.md
│   │   ├── docker-compose.yml
│   │   ├── package.json
│   │   ├── src
│   │   ├── tsconfig.build.json
│   │   └── tsconfig.json
│   ├── 08-webpack
│   │   ├── .eslintrc.js
│   │   ├── .gitignore
│   │   ├── nest-cli.json
│   │   ├── package.json
│   │   ├── src
│   │   ├── tsconfig.build.json
│   │   ├── tsconfig.json
│   │   └── webpack-hmr.config.js
│   ├── 09-babel-example
│   │   ├── .babelrc
│   │   ├── .gitignore
│   │   ├── index.js
│   │   ├── jsconfig.json
│   │   ├── nodemon.json
│   │   ├── package.json
│   │   └── src
│   ├── 10-fastify
│   │   ├── .eslintrc.js
│   │   ├── .gitignore
│   │   ├── README.md
│   │   ├── package.json
│   │   ├── src
│   │   ├── tsconfig.build.json
│   │   └── tsconfig.json
│   ├── 11-swagger
│   │   ├── .eslintrc.js
│   │   ├── .gitignore
│   │   ├── README.md
│   │   ├── nest-cli.json
│   │   ├── package.json
│   │   ├── src
│   │   ├── tsconfig.build.json
│   │   └── tsconfig.json
│   ├── 12-graphql-schema-first
│   │   ├── .eslintrc.js
│   │   ├── .gitignore
│   │   ├── README.md
│   │   ├── generate-typings.ts
│   │   ├── nest-cli.json
│   │   ├── package.json
│   │   ├── src
│   │   ├── tsconfig.build.json
│   │   └── tsconfig.json
│   ├── 13-mongo-typeorm
│   │   ├── .eslintrc.js
│   │   ├── .gitignore
│   │   ├── README.md
│   │   ├── docker-compose.yml
│   │   ├── package.json
│   │   ├── src
│   │   ├── tsconfig.build.json
│   │   └── tsconfig.json
│   ├── 14-mongoose-base
│   │   ├── .eslintrc.js
│   │   ├── .gitignore
│   │   ├── README.md
│   │   ├── docker-compose.yml
│   │   ├── package.json
│   │   ├── src
│   │   ├── tsconfig.build.json
│   │   └── tsconfig.json
│   ├── 15-mvc
│   │   ├── .eslintrc.js
│   │   ├── .gitignore
│   │   ├── nodemon.json
│   │   ├── package.json
│   │   ├── public
│   │   ├── src
│   │   ├── tsconfig.build.json
│   │   ├── tsconfig.json
│   │   └── views
│   ├── 16-gateways-ws
│   │   ├── .eslintrc.js
│   │   ├── .gitignore
│   │   ├── client
│   │   ├── package.json
│   │   ├── src
│   │   ├── tsconfig.build.json
│   │   └── tsconfig.json
│   ├── 17-mvc-fastify
│   │   ├── .eslintrc.js
│   │   ├── .gitignore
│   │   ├── README.md
│   │   ├── nodemon.json
│   │   ├── package.json
│   │   ├── public
│   │   ├── src
│   │   ├── tsconfig.build.json
│   │   ├── tsconfig.json
│   │   └── views
│   ├── 18-context
│   │   ├── .eslintrc.js
│   │   ├── .gitignore
│   │   ├── package.json
│   │   ├── src
│   │   ├── tsconfig.build.json
│   │   └── tsconfig.json
│   ├── 19-auth-jwt
│   │   ├── .eslintrc.js
│   │   ├── .gitignore
│   │   ├── e2e
│   │   ├── nest-cli.json
│   │   ├── nodemon-debug.json
│   │   ├── nodemon.json
│   │   ├── package.json
│   │   ├── src
│   │   ├── tsconfig.build.json
│   │   └── tsconfig.json
│   ├── 20-cache
│   │   ├── .eslintrc.js
│   │   ├── .gitignore
│   │   ├── jest.json
│   │   ├── package.json
│   │   ├── src
│   │   ├── tsconfig.build.json
│   │   └── tsconfig.json
│   ├── 21-serializer
│   │   ├── .eslintrc.js
│   │   ├── .gitignore
│   │   ├── jest.json
│   │   ├── package.json
│   │   ├── src
│   │   ├── tsconfig.build.json
│   │   └── tsconfig.json
│   ├── 22-graphql-prisma
│   │   ├── .eslintrc.js
│   │   ├── .gitignore
│   │   ├── .graphqlconfig.yml
│   │   ├── database
│   │   ├── nodemon.json
│   │   ├── package.json
│   │   ├── src
│   │   ├── tsconfig.build.json
│   │   └── tsconfig.json
│   ├── 23-graphql-code-first
│   │   ├── .eslintrc.js
│   │   ├── .gitignore
│   │   ├── nest-cli.json
│   │   ├── package.json
│   │   ├── schema.gql
│   │   ├── src
│   │   ├── tsconfig.build.json
│   │   └── tsconfig.json
│   ├── 24-serve-static
│   │   ├── .eslintrc.js
│   │   ├── .gitignore
│   │   ├── client
│   │   ├── package.json
│   │   ├── src
│   │   ├── tsconfig.build.json
│   │   └── tsconfig.json
│   ├── 25-dynamic-modules
│   │   ├── .eslintrc.js
│   │   ├── .gitignore
│   │   ├── .prettierrc
│   │   ├── README.md
│   │   ├── config
│   │   ├── nest-cli.json
│   │   ├── package.json
│   │   ├── src
│   │   ├── tsconfig.build.json
│   │   └── tsconfig.json
│   ├── 26-queues
│   │   ├── .eslintrc.js
│   │   ├── .gitignore
│   │   ├── .prettierrc
│   │   ├── README.md
│   │   ├── docker-compose.yml
│   │   ├── nest-cli.json
│   │   ├── package.json
│   │   ├── src
│   │   ├── tsconfig.build.json
│   │   └── tsconfig.json
│   ├── 27-scheduling
│   │   ├── .eslintrc.js
│   │   ├── .gitignore
│   │   ├── .prettierrc
│   │   ├── README.md
│   │   ├── nest-cli.json
│   │   ├── package.json
│   │   ├── src
│   │   ├── tsconfig.build.json
│   │   └── tsconfig.json
│   ├── 28-sse
│   │   ├── .eslintrc.js
│   │   ├── .gitignore
│   │   ├── .prettierrc
│   │   ├── README.md
│   │   ├── jest.json
│   │   ├── nest-cli.json
│   │   ├── package.json
│   │   ├── src
│   │   ├── tsconfig.build.json
│   │   └── tsconfig.json
│   ├── 29-file-upload
│   │   ├── .eslintrc.js
│   │   ├── .gitignore
│   │   ├── README.md
│   │   ├── e2e
│   │   ├── jest.json
│   │   ├── package.json
│   │   ├── src
│   │   ├── tsconfig.build.json
│   │   └── tsconfig.json
│   ├── 30-event-emitter
│   │   ├── .eslintrc.js
│   │   ├── .gitignore
│   │   ├── .prettierrc
│   │   ├── nest-cli.json
│   │   ├── package.json
│   │   ├── src
│   │   ├── test
│   │   ├── tsconfig.build.json
│   │   └── tsconfig.json
│   ├── 31-graphql-federation-code-first
│   │   ├── README.md
│   │   ├── gateway
│   │   ├── posts-application
│   │   └── users-application
│   └── 32-graphql-federation-schema-first
│       ├── README.md
│       ├── gateway
│       ├── posts-application
│       └── users-application
├── scripts
│   ├── prepare.sh
│   ├── run-integration.sh
│   ├── test.sh
│   └── update-samples.sh
├── tools
│   ├── benchmarks
│   │   ├── check-benchmarks.ts
│   │   ├── get-benchmarks.ts
│   │   └── report-contents.md
│   └── gulp
│       ├── config.ts
│       ├── gulpfile.ts
│       ├── tasks
│       ├── tsconfig.json
│       └── util
├── tsconfig.json
├── tsconfig.spec.json
└── yarn.lock

221 directories, 361 files
```


### 👨🏻‍💻 common

学到啥
1. 统一导出 index.ts，按目录的顺序统一导出。
2. constants.ts 常量定义。 `__httpCode__`
3. 根据 export enum HttpStatus {  处理相关的异常

### 👨🏻‍💻 core
#### adapters
> 实现 http 适配器 AbstractHttpAdapter。 common 层定义、类型调用。

```tslang
export abstract class AbstractHttpAdapter<
  TServer = any,
  TRequest = any,
  TResponse = any,
> implements HttpServer<TRequest, TResponse>
{
```


`export class ExpressAdapter extends AbstractHttpAdapter {`

```tslang
import { NestExpressApplication } from '@nestjs/platform-express';

    const app = await NestFactory.create<NestExpressApplication>(AppModule, {
      abortOnError: false,
    });
```

## ❓疑问记录
### 1、peerDependenciesMeta 的作用

### 2、reflect-metadata 作用

> TODO 深入补充了解。

控制反转和依赖注入。

```tslang
type Constructor<T = any> = new (...args: any[]) => T;

const Injectable = (): ClassDecorator => target => {};

class OtherService {
  a = 1;
}

@Injectable()
class TestService {
  constructor(public readonly otherService: OtherService) {}

  testMethod() {
    console.log(this.otherService.a);
  }
}

const Factory = <T>(target: Constructor<T>): T => {
  // 获取所有注入的服务
  const providers = Reflect.getMetadata('design:paramtypes', target); // [OtherService]
  const args = providers.map((provider: Constructor) => new provider());
  return new target(...args);
};

Factory(TestService).testMethod(); // 1
```

- https://jkchao.github.io/typescript-book-chinese/tips/metadata.html#%E8%87%AA%E5%AE%9A%E4%B9%89-metadatakey Reflect Metadata
-  https://segmentfault.com/a/1190000008626680 Angular 2 DI - IoC & DI - 1

### 3、uuid 生成的作用

### 4、如何处理 Error 的循环依赖

### 5、. - 文件命名的规则
### 6、axios 实例默认集成，无需自己 provider

## TODOs
- [ ] reflect-metadata 知识充电
- [ ] rxjs 知识充电
- [ ] express 知识充电
- [ ] IoC、DI 知识充电
- [ ] Reflect、Decorator 等 ESx 知识充电
- [ ] HTTP CODE 知识充电
- [ ] 核心库知识充电
- [ ] node tree 知识充电
- [ ] 设计模式知识充电
- [ ] npm 钩子知识充电
- [ ] 单元测试 chai mocha 等知识充电