# ajax拦截器

> 在ajax请求then或catch之前进行拦截操作

```javascript
import axios from 'axios'
class httpRequest {
    constructor () {
        this.baseURL = this.process.env.NODE_ENV === 'development' ? 							'http://localhost:8080' : '/'
        this.timeout = 2000
    },
    request(){
        const http = axios.create({
            baseURL: this.baseURL,
            timeout: this.timeout
        })
        http.interceptors.request.use((config) => {
            // 进行请求发送拦截后的操作
            return config
        }，err => {
            Promise.reject(err) 
        })
        http.interceptors.response.use((data) => {
            // 进行请求响应拦截后的操作
            return data
        }，err => {
            Promise.reject(err) 
        })
    }    
}
```

