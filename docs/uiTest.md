# UI测试

## e2e测试

RA使用 [jest](https://github.com/facebook/jest) + [puppeteer](https://github.com/puppeteer/puppeteer)  的端到端测试方案，配置文件位于 `jest-puppeteer.config.js` 与 `jest.config.js`

> 默认会跳过安装chromium 如果需要安装chromium请运行此命令 `node node_modules/puppeteer/install.js`

### 写一个 e2e 用例

假设有一个需求，用户在登录页面输入错误的用户名和密码，点击登录后，出现错误提示框。

我们写一个用例来保障这个流程。在 src/e2e/ 目录下建一个 login.e2e.js 文件，按上述业务需求描述测试用例。

```javascript
const BASE_URL = `http://localhost:${process.env.PORT || 9527}`;

describe('login', () => {
  beforeAll(async () => {
    jest.setTimeout(1000000);
  });

  beforeEach(async () => {
    await page.goto(`${BASE_URL}/#/user/login`, {
      waitUntil: 'networkidle2'
    });
  });

  afterEach(async () => {
    await page.waitFor(1000);
  });

  it('should login with success', async () => {
    await page.waitForSelector('#normal_login_username', {
      timeout: 2000
    });
    await page.type('#normal_login_username', 'admin');
    await page.type('#normal_login_password', '123');
    await page.click('button[type="submit"]');
    await page.waitForSelector('.ant-message-success');
  });

  it('should login with failure', async () => {
    await page.waitForSelector('#normal_login_username', {
      timeout: 2000
    });
    await page.type('#normal_login_username', 'wrong_user');
    await page.type('#normal_login_password', 'wrong_password');
    await page.click('button[type="submit"]');
    await page.waitForSelector('.ant-alert-error');
  });
});

```


### 执行e2e测试

使用以下的命令将统一搜索和执行 `src/e2e` 下 `*.e2e.js` 命名的用例文件。

```bash
yarn test:e2e

yarn run v1.13.0
$ jest -c e2e/jest.config.js
 PASS  e2e/404.e2e.js (8.564s)
  404
    ✓ test route 404 (435ms)

 PASS  e2e/login.e2e.js (11.638s)
  login
    ✓ should login with success (4167ms)
    ✓ should login with failure (1826ms)

 PASS  e2e/skeleton.e2e.js (35.626s)
  mainLayout
    ✓ test pages / (1857ms)
    ✓ test pages /dashboard (1014ms)
    ✓ test pages /program/analysis (1544ms)
    ✓ test pages /program/monitor (1790ms)
    ✓ test pages /program/platform (1360ms)
    ✓ test pages /program/unit (1600ms)
    ✓ test pages /form/basicForm (1341ms)
    ✓ test pages /form/stepForm (1148ms)
    ✓ test pages /form/test/test1 (1077ms)
    ✓ test pages /list/basicList (1150ms)
    ✓ test pages /list/cardList (1082ms)
    ✓ test pages /list/basicTable (1120ms)
    ✓ test pages /map (1099ms)
    ✓ test pages /gallery (1503ms)
    ✓ test pages /result/successResult (1121ms)
    ✓ test pages /result/failedResult (1080ms)
    ✓ test pages /exception/403 (1052ms)
    ✓ test pages /exception/404 (1059ms)
    ✓ test pages /exception/500 (1050ms)
    ✓ test pages /exception/home (1067ms)

Test Suites: 3 passed, 3 total
Tests:       23 passed, 23 total
Snapshots:   0 total
Time:        36.381s
Ran all test suites.
✨  Done in 40.66s.
```

> 需要注意的是，执行e2e测试时，需要启动前端服务。


## unit单元测试

单元测试用于测试 React UI 组件的表现，我们使用 jest 作为测试框架。

jest 是一个 node 端运行的测试框架，使用了 [jsdom](https://github.com/jsdom/jsdom) 来模拟 DOM 环境，适合用于快速测试 React 组件的逻辑表现，需要真实浏览器可以参考 E2E 测试部分。

### 写一个用例
比如，我们可以建一个文件 src/components/HighLight/index.test.js 来测试高亮组件是否满足UI预期。

```javascript
import React from 'react';
import { mount } from 'enzyme';
import HighLight from '../HighLight'; // 引入对应的 React 组件

describe('Highlight', () => {
  const Element =
    '<span style="color: rgb(30, 144, 255);">2333</span><span>woasd</span><span style="color: rgb(30, 144, 255);">2333</span>';  // 期望的HTML内容

  beforeAll(async () => {
    jest.setTimeout(1000000);
  });

  it('highlight text', async () => {
    const wrapper = mount(<HighLight val="2333woasd2333" tarVal="2333" />);  // 进行渲染
    expect(wrapper.html()).toEqual(Element);  // render结果符合期望内容
  });
});
```

这里使用了 [enzyme](https://enzymejs.github.io/enzyme/) 作为测试库，它提供了大量实用的 API 来帮助我们测试 React 组件。断言部分沿用了 jest 默认的 [jasmine2 expect 语法](https://jestjs.io/docs/en/expect.html#content)。

### 执行测试

使用以下的命令将统一搜索和执行 `src` 下 `*.test.js` 命名的用例文件。

```bash
yarn test:unit

yarn run v1.13.0
$ jest -c jest.config.js
 PASS  src/components/Authorized/checkpermission.test.js (5.065s)
  test CheckPermissions
    ✓ Correct string permission authentication (2ms)
    ✓ Correct string permission authentication (1ms)
    ✓ authority is undefined , return ok
    ✓ currentAuthority is undefined , return error
    ✓ Wrong string permission authentication
    ✓ Correct Array permission authentication
    ✓ Wrong Array permission authentication,currentAuthority error
    ✓ Wrong Array permission authentication
    ✓ authority is string, currentAuthority is array, return ok (1ms)
    ✓ authority is string, currentAuthority is array, return ok
    ✓ authority is array, currentAuthority is array, return ok
    ✓ authority is undefined , return ok

 PASS  src/components/HighLight/highlight.test.js (5.237s)
  Highlight
    ✓ highlight text (51ms)

 PASS  src/components/Layout/BasicLayout/basicLayout.test.js (10.494s)
  BasicLayout
    ✓ BasicLayout change header (144ms)
    ✓ BasicLayout change siderBar (23ms)
    ✓ BasicLayout change siteLogo (43ms)

Test Suites: 3 passed, 3 total
Tests:       16 passed, 16 total
Snapshots:   0 total
Time:        11.37s, estimated 14s
Ran all test suites.
✨  Done in 14.19s.
```