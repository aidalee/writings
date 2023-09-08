> axios 封装 in request.ts

```js
import axios from 'axios'

import type { AxiosInstance, AxiosRequestConfig, AxiosResponse, InternalAxiosRequestConfig } from 'axios'

import { message } from 'antd'

import envConfig from "../../config"

const MODE = import.meta.env.MODE

type Result<T> = {
  code: number
  message: string
  data: T
}

class Request {
  instance: AxiosInstance
  baseConfig: AxiosRequestConfig = {
    baseURL: envConfig[MODE].apiBaseUrl,
    timeout: 5000
  }
  constructor(config: AxiosRequestConfig={}) {
    this.instance = axios.create(Object.assign(this.baseConfig, config))

    this.instance.interceptors.request.use(
      (config: InternalAxiosRequestConfig) => {
        // 配置headers等
        return config
      },
      (error: any)=>{
        return Promise.reject(error)
      }
    )

    this.instance.interceptors.response.use(
      (res: AxiosResponse) => {
        return res?.data || true
      },
      (error: any)=>{
        let msg = ''
        switch (error.response.status) {
          case 401:
            msg = '未授权'
            break;
          case 403:
            msg = "拒绝访问(403)";
            break;
          case 404:
            msg = "请求出错(404)";
            break;
          case 500:
            msg = "服务器错误(500)";
            break;
          default:
            msg = `请求出错(${error.response.status})!`
        }

        message.error(msg)
        return Promise.reject(error.response)

      }
    )
  }


  public request(config: AxiosRequestConfig): Promise<AxiosResponse> {
    return this.instance.request(config)
  }

  public get<T = any>(
    url: string,
    config?: AxiosRequestConfig
  ): Promise<AxiosResponse<Result<T>>> {
    return this.instance.get(url, config)
  }

  public post<T = any>(
    url: string,
    data?: any,
    config?: AxiosRequestConfig
  ): Promise<AxiosResponse<Result<T>>> {
    return this.instance.post(url,data,config)
  }
  
  public put<T = any>(
    url: string,
    data?: any,
    config?: AxiosRequestConfig
  ): Promise<AxiosResponse<Result<T>>> {
    return this.instance.put(url,data,config)
  }

  public delete<T = any>(
    url: string,
    config?: AxiosRequestConfig
  ): Promise<AxiosResponse<Result<T>>> {
    return this.instance.delete(url, config)
  }

}

export default new Request()
```

> 不同环境接口域名配置 in config.ts

```js
interface EnvConfigType {
  [key: string]: {
    apiBaseUrl: string
  }
}


const envConfig: EnvConfigType = {
  development: {
    apiBaseUrl: "", // 开发环境接口请求
  },
  test: {
    apiBaseUrl: "", // 测试环境接口地址
  },
  production: {
    apiBaseUrl: "", // 正式环境接口地址
  }
};

export default envConfig
```

> 使用示例 in auth.ts

```js
import request from '@/utils/request'
interface IParams {
  username: string
  password: string
}

export const httpLogin = <T, U extends IParams>(params: U) => request.post<T>(`/api/authentication`, params)
```