# React-全家桶仿简书部分功能

### [前言](_)

> * 这段时间项目重构，斟酌良久选择了React。其原因1.前段时间接触过React，对jsx语法有着很大的兴趣。2.React的灵活性要比Vue写起来舒服，没有那么多限制。3.总是要尝试新的东西，不能将自己老是放在舒适区里。
* 主要用到了React+React-Router4+Ant-design+Mobx+React-lazyload等一些其它依赖。

### `技术栈以及组件库`
* React-Router4：react路由
* Ant-design：虽然前段时间Ant彩蛋事件,但是对比了其它框架总是觉得有的太重，有的相对而言国内生态不是很好，所以还是选用了Ant-design
* Mobx：由于Redux的繁琐，及考虑到其他同事上手项目的速度，选用了Mobx作为状态管理。
* React-lazyload：路由分割，但是感觉有坑，在切换路由时会有闪烁问题，体验不是太好。
* Sass：主要用于封装公共颜色及公共方法Mixins库，加上写法个人也比较喜欢。
* Axios：封装Axios公共方法及一些拦截器相应超时等。
* Animate：由于太懒了,就引用了css动画库

### `文件结构`
```
┣━ build   // 打包文件
┣━ public  
┣━ src //开发目录
  ┣━ common   //公用组件
    ┣━ header   //头部组件
    ┣━ footer   //页足组件
    ┣━ loading   //公共loading组件
    ┣━ userCenter   //公共用户信息组件
  ┣━ pages   //页面
    ┣━ home   //主页
    ┣━ ...    //等二十多页面...
  ┣━ statics   //静态文件
    ┣━ css  //动画库等
    ┣━ image   //本地图片
    ┣━ js   //api接口、axios封装、公共Js方法封装等...
  ┣━ store   //Mobx数据
    ┣━ ...
  ┣━ Mixins   //sass公共库
  ┣━ App.js   //入口及路由
  ┣━ index.js   //js文件入口
  ┣━ reset   //样式重置
┣━ .gitignore   //git忽略上传文件
┣━ package.json   //模块的描述文件
┣━ README.md   //说明文件
┣━ yarn.lock   //模块的描述文件
```

### `实现主要几个功能`

* **Mixins库封装**

将公共颜色及一些公共的css方法封装在Mixins库中，在我们使用box-shadow、transition等css3浏览器兼容性属性时没必要去一个个添加兼容。以及根据用户Vip等级去换肤等功能...

```
//公共颜色
$orange: #ff8d6b;      //副色调
$white: #ffffff;       //白色
$black: #000000;       //黑色
$lightWhite: #E7F0FF;  //浅白色
$blue: #4C82DF;        //副色调
$darkBlue: #364d79;    //副深主色
$textColor: #B2B2B2;   //文本颜色
$darkGrey: #313234;    //深灰色
$lightGrey: #D0D0D0;   //浅灰色
$mainCyan:#EEF5FD;     //主色调
$appleRed: #e9552b;    //苹果红
$pink: #f97695;        //粉色


//公共方法
@mixin border-radius($radius) { //圆角
-webkit-border-radius: $radius;
 -moz-border-radius: $radius;
  -ms-border-radius: $radius;
      border-radius: $radius;
}

@mixin abs-pos($top: auto, $right: auto, $bottom: auto, $left: auto) { //绝对定位
top: $top;
right: $right;
bottom: $bottom;
left: $left;
position: absolute;
}

@mixin transition($times) { //过渡效果
-webkit-transition: all $times ease;
 -moz-transition: all $times ease;
  -ms-transition: all $times ease;
      transition: all $times ease;
}
...太长了就不展示了,这里参考了挺多前辈的封装,然后根据项目的实际需求去进行了封装。
```

* **Axios封装**

这里对Axios的拦截器,连接超时,Post方法等进行了封装，更加方便的去调用。由于用户中心那里调用了大量的接口,而且是并发所以会导致在切换下个路由时还有一些接口在padding，会影响到下个路由的性能，尝试使用了Axios的cancelToken取消请求，但是效果不是太理想，后将用户中心的接口更改为链式回调的方法解决了这个坑。

```
//POST传参序列化(请求拦截器)
axios.interceptors.request.use((config) => {
   //在发送请求之前做某件事
   if(config.method  === 'post'){
       // if(config.headers["Content-Type"]!="multipart/form-data"){
           if (sessionStorage.getItem("authorization")) {
             config.headers.common['Authorization'] = sessionStorage.getItem("authorization")
           }
       let str = [];
       let sign =[];
           for(var p in config.data) {
         sign.push(p+config.data[p]);
         str.push(encodeURIComponent(p) + "=" + encodeURIComponent(config.data[p]));
       }
       let time =Math.round(new Date().getTime()/1000).toString();
       sign.push("timestamp"+time);
       str.push(encodeURIComponent("timestamp") + "=" + encodeURIComponent(time));
       str.push(encodeURIComponent("sign") + "=" + encodeURIComponent(md5("jXPHEED7kAo1SQ"+sign.sort().join("")).toUpperCase()));
         config.data = str.join("&");
       }
   // }
 return config;
},(error) =>{
   return Promise.reject(error);
});
//返回状态判断(响应拦截器)
axios.interceptors.response.use((res) =>{
   if(res.headers.authorization!=null){
       sessionStorage.setItem("authorization",res.headers.authorization)
 }
 if(!res.data.success){
   return Promise.resolve(res);
 }
 return res;
},(error) => {
   return Promise.reject(error);
});

//post请求封装
export const POST=(url, params)=>{
 return new Promise((resolve, reject) => {
       axios.post(url,params).then(response => {
     if(response.data.code===200){
               resolve(response.data);
           }else{
               if(response.data.message!=null&&response.data.message!=""){
         reject(response.data);
         if(res.data.message=='Unauthenticated'){
           setTimeout(() => {
                           // window.location.href = 'http://lw.0731333.com/smart';
                       }, 2000);
         }
               }else{
                   resolve(response.data);
               }
           }

           resolve(response.data);
       },err=>{
     reject(err);
   })
       .catch((error) => {
           reject(error)
       })
   })
};
```

* **状态管理数据**

在这里将一些公共的数据使用Mobx进行统一的存储。使用起来比较方便。
这里要注意在index.js入口文件将存储的数据进行统一的注入。
```
<Provider stores={stores}>
    <App />
</Provider>,
```
具体单独的组件的调用Mobx的方法有两种方法，由于比较懒就是用了es7@装饰器语法进行注入,这里和React-router4注入方法类似
```
    import { observer, inject }  from 'mobx-react'
    @inject('stores')
    @observer
    class Header extends Component {
        console.log(this.props.stores.userInfo.user_nicename) //获取数据
        this.props.stores.loginStatu(2) //触发action方法等...
    }
```
mobx数据文件的简单使用：
```
import { observable, action } from 'mobx';

class Login {
  @observable footer = 1; //底部状态
  @observable loginStatus = false; //登录状态
  @observable userInfo = []; //用户信息

  @action
  changeFooter(type) {//离开一屏页面切换回底部
    this.footer = type;
  }

  @action
  loginStatu(type, info) {//存储用户信息
    if(type === 1) {
      this.loginStatus = true;
      this.userInfo = info;
    }else if(type === 2) {
      this.loginStatus = false;
    }
  }

}

const stores = new Login()
```
...由于项目功能太多就不一一展示了。

### `主要解决的坑`
* **当前路由页进行异步操作造成内容泄露警告：**

当前路由页进行异步操作的接口返回时间过慢,当数据没有返回但是在异步方法中调用了setstate方法就会造成内存泄露。这里参考了挺多国内文章，只能说比较坑，没有说到关键上。后来google到一篇文档讲解的很详细。
这里解释了原因：
```
The shown warning(s) usually show up when this.setState() is called in a component even though the component got already unmounted. The unmounting can happen for different cases
//附链接（https://www.robinwieruch.de/react-warning-cant-call-setstate-on-an-unmounted-component/）
```
解决方法:
```
//you can use this knowledge not to abort the request itself, but avoid calling this.setState() on your component instance, even though the component already unmounted. It will prevent the warning.
class News extends Component {
    _isMounted = false;

    constructor(props) {
    super(props);

    this.state = {
      news: [],
    };
    }

    componentDidMount() {
    this._isMounted = true;

    axios
      .get('https://hn.algolia.com/api/v1/search?query=react')
      .then(result => {
        if (this._isMounted) {
          this.setState({
            news: result.data.hits,
          });
        }
      });
    }

    componentWillUnmount() {
    this._isMounted = false;
    }

    render() {
    ...
    }
}
```

* **链式异步回调**

由于用户中心那里调用了大量的接口,而且是并发所以会导致在切换下个路由时还有一些接口在padding，会影响到下个路由的性能，尝试使用了Axios的cancelToken取消请求，但是效果不是太理想，后将用户中心的接口更改为链式回调的方法解决了这个坑。
```
refreshAmountLoad(code) {
    let val = this.state.options;
    let _this = this;
    if(code == 0) {
      this.refreshAmountLoad(1);
      return false;
    }
    if(code >= val.length) {
      return false;
    }
    PORT.POST(PORT.API.updateMoneyUrl, {
      game: val[code].type
    } , function(msg){
      if(msg.status == 1) {
        if(_this._isMounted) {
          for(let item in val) {
            if(val[item].type == val[code].type) {
              val[item].loading = false;
              val[item].money = msg.data[`${val[code].type}_money`];
            }
          }
          _this.setState({options:[...val]});
          code++;
          _this.refreshAmountLoad(code);
        }
      }else if(msg.status == -1) {
        message.warning('...');
        for(let item in val) {
          val[item].loading = false;
        }
        _this.setState({
          options: [...val]
        })
      }else if(msg.status == -2) {
        if(_this._isMounted) {
          message.warning(msg.info);
          for(let item in val) {
            val[item].loading = false;
          }
          _this.setState({
            options: [...val]
          })
        }
      }else {
        if(_this._isMounted) {
          _this.refreshAmountLoad(code)
          message.warning(msg.info)
        }
      }
    })
  }
```

其实开发中踩坑无数，但是太多了，就不进行列举了，如果你有问题欢迎和我交流讨论。

### `结语`
> 由于工作比较忙,所以文档也不是很完善，整理的也不太好。如果有什么疑问，或者遇到和我相同的问题，欢迎和我交流讨论。

> 当然代码还有很多不足之处，我也在不断地进步。有什么不足之处还请指出。但是如果你喷我！我就骂你！！！
