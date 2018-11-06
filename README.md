## 所用技术栈
1.vue <br />
2.scss

## 该组件使用文档
1.change 事件，当改变后获取到地址ID数据，和地址名称数组，即change(array, array), 第一个参数为选中的地址ID数组，第二个参数为选中的地址名数组<br />
使用：1：引入组件 import myAddress  from '@/components/address/addressComponent' （根据自己存放的位置引入)<br />
2：注入组件components: { myAddress } <br />
3： <my-address :addressCode="addressCode" @change="getSelectAddress"></my-address> <br />
4： getSelectAddress(ids, names){ <br />
       //ids为地址ID数组
       //names为地址名数组
    }<br />
 

## 适用范围
1.获取省份，城市，区，乡镇的接口必须是独立的 <br />
2.数据源是一个对象，如下图<br/>
[https://github.com/wsq815/vue-JD-address/blob/master/01.png](https://github.com/wsq815/vue-JD-address/blob/master/01.png)<br />

## 效果视频
1.[https://github.com/wsq815/vue-JD-address/blob/master/0935c37b0f0cb888f3cfae6c47e779bb.mp4](https://github.com/wsq815/vue-JD-address/blob/master/0935c37b0f0cb888f3cfae6c47e779bb.mp4)
