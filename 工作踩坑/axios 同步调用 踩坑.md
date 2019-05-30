# axios 同步调用 踩坑

### 场景：需要ajax请求后更新某个全局变量，将此全局变量赋值后再通过该全局变量的值作为参数传到后台拿取所要的数据
**axios是不支持同步调用的**，
在这里我们可以使用 promise 编程方式来实现，例子如下：

```javascript
                getQueues: function () {
                    axios.get('http://localhost:9091/getQueues').then(res => {
                        res.data.forEach(item => {
                            var queueBasicInfo = {
                                destinationName:item.destinationName,
                                brokerName:item.brokerName
                            }
                            this.queueBasicInfos.push(queueBasicInfo);
                        })
                    return this.queueBasicInfos;
                }).then(data => {
                        this.getMonitorDestinationData(data)
                    })
                }
```
> 这里可以使用 then().then()... 这种方式来实现，这样就可以拿到值后再往下执行。


