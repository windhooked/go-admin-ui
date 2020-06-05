# go-admin-ui

![build](https://github.com/wenjianzhang/go-admin-ui/workflows/build/badge.svg?branch=master) ![MIT](https://img.shields.io/github/ license/mashape/apistatus.svg)

## Introduction

[goadmin-ui](https://github.com/wenjianzhang/go-admin-ui) is a front-end implementation of a background basic module, which is based on [vue-element-admin](https://panjiachen.github. io/vue-element-admin). It implements a set of basic modules of the most basic background authority management front end. You can quickly open your own project, regardless of how to implement the permission function and how to design the operation page. Here is basically satisfied, I hope to provide convenience for everyone.

-[Online Preview] (http://www.zhangwj.com/#/login)

-[Backend Project] (https://github.com/wenjianzhang/go-admin)

-[Front End Project] (https://github.com/wenjianzhang/go-admin-ui)


## Features

```
- log in
- drop out

-Home
- System Management
   - User Management
   -Menu management
   -Role management
   -Department management
   -Post management
   -Dictionary management
   - parameter settings
   -Log management
     -Login log

- basic information
   -Excel

- System Tools
   -System interface
   -Calendar

```

## Development

```bash
# Clone project
git clone https://github.com/wenjianzhang/go-admin-ui.git

# Enter the project directory
cd go-admin-ui

# Install dependencies
npm install

# It is recommended not to directly use cnpm to install dependencies, as there will be various weird bugs. The problem of slow npm download speed can be solved by the following operations
npm install --registry=https://registry.npm.taobao.org

# Start the service
npm run dev
```

Browser visit http://localhost:9527

## Post

```bash
# Build test environment
npm run build:stage

# Build production environment
npm run build:prod
```

## Other

```bash
# Preview release environment effect
npm run preview

# Preview release environment effect + static resource analysis
npm run preview - --report

# Code format check
npm run lint

# Code format check and automatic repair
npm run lint - --fix
```

## online preview

[Online Demo](http://www.zhangwj.com/#/login)

## License

[MIT](https://github.com/wenjianzhang/go-admin-ui/blob/master/LICENSE)

Copyright (c) 2020 wenjianzhang
