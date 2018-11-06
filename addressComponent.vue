<template>
    <div class="address-select-wrapper">
        <div class="my-input my-address" @click="selectAddress">
            <span>所在地址&nbsp;&nbsp;&nbsp;{{address}}</span>
            <span class="my-address-right">请选择</span>
        </div>
        <div class="w-addres-mask" v-if="showSelectAddress"></div>
        <div class="w-address-bg"  @click="selectAddress">
            <transition name="fold"> 
                    <div class="w-address-content" @click.stop v-if="showSelectAddress">
                        <div class="content-header">
                            <span>配送至</span>
                            <i class="close-w-address" @click="selectAddress"><a href="javascript:void(0)" class="close"></a></i>
                        </div>
                        <div class="content-body">
                            <div class="selected-list">
                                <div class="selected-list-item" :class="provinceActive ? 'active-item' : ''" @click="getProvinces">{{province ? province : '请选择'}}</div>
                                <div class="selected-list-item" :class="cityActive ? 'active-item' : ''" @click="toggleCity">{{city ? city : JSON.stringify(citysList) != '{}' ? '请选择': ''}}</div>
                                <div class="selected-list-item" :class="districtActive ? 'active-item' : ''" @click="toggleDistrice">{{district ? district : JSON.stringify(districtList) != '{}' ? '请选择': ''}}</div>
                                <div class="selected-list-item" :class="townActive ? 'active-item' : ''" @click="toggleTown">{{town ? town : JSON.stringify(townList) != '{}' ? '请选择': ''}}</div>
                            </div>
                            <div class="w-area">
                                <ul class="w-area-list">
                                    <li class="area-list-item" v-show="showProvinces" :class="province == label ? 'active-item' : ''"
                                        @click="selectProvinceItem(value, label)" v-for="(value, label) in provincesList" :key="value">
                                        {{label}}
                                    </li>
                                    <li class="area-list-item" v-show="showcitys" :class="city == label ? 'active-item' : ''"
                                        @click="selectCityItem(value, label)" v-for="(value, label) in citysList" :key="value">
                                        {{label}}
                                    </li>
                                    <li class="area-list-item" v-show="showDistrict" :class="district == label ? 'active-item' : ''"
                                        @click="selectDistrictItem(value, label)" v-for="(value, label) in districtList" :key="value">
                                        {{label}}
                                    </li>
                                    <li class="area-list-item" v-show="showTown" :class="town == label ? 'active-item' : ''"
                                        @click="selectTown(value, label)" v-for="(value, label) in townList" :key="value">
                                        {{label}}
                                    </li>
                                </ul>
                            </div>
                        </div>
                    </div>
            </transition>            
        </div>
    </div>
</template>
<script>
    import {getAllProvince, getCityByProvinceId, getAreaByCityId, getTownByAreaId} from '@/api/address';
    export default {
        props: {
            addressCode: Array
        },
        data() {
            return {
                //省市区镇列表
                provincesList: {},
                citysList: {},
                districtList: {},
                townList: {},
                //省市区镇文字
                province: '',
                city: '',
                district: '',
                town: '',
                //省市区ID
                provinceId: '',
                cityId: '',
                districtId: '',
                townId: '',
                //展示省市区镇列表
                showProvinces: false,
                showcitys: false,
                showDistrict: false,
                showTown: false,
                //设置选中时省市区镇高亮样式
                provinceActive: false,
                cityActive: false,
                districtActive: false,
                townActive: false,
                ids:[],
                names: [],
                showSelectAddress: false,
                address: ''
            }
        },
        created(){
            this.getProvinces();
        },
        methods: {
            async getProvinces(){
                const res = await getAllProvince();
                this.provincesList = res.data.result;
                this.showProvinces = true;
                this.showcitys = false;
                this.showDistrict=false;
                this.showTown=false;

                this.provinceActive = true;
                this.cityActive = false;
                this.districtActive = false;
                this.townActive = false;
            },
            async selectProvinceItem(value, label){
                this.province = label;
                this.provinceId = value;
                const res = await getCityByProvinceId({provinceId: value});
                this.citysList = res.data.result;
                this.showProvinces = false;
                this.showcitys = true;
                this.showDistrict=false;
                this.showTown=false;

                this.provinceActive = false;
                this.cityActive = true;
                this.districtActive = false;
                this.townActive = false;

                this.city = '' //清空城市
                this.district = '' //清空区
                this.town = '' //清空乡镇
                this.districtList = {}; //清空区列表
                this.townList = {}; //清空乡镇列表
            },
            async selectCityItem(value, label){
                this.city = label;
                this.cityId = value;
                const res = await getAreaByCityId({cityId: value});
                console.log(res, '区列表');
                this.districtList = res.data.result;

                this.showProvinces = false;
                this.showcitys = false;
                this.showDistrict= true;
                this.showTown=false;

                this.provinceActive = false;
                this.cityActive = false;
                this.districtActive = true;
                this.townActive = false;
                this.district = '' //清空区
                this.town = '' //清空乡镇
                this.townList = {}; //清空乡镇列表
            },
            async selectDistrictItem(value, label){
                this.district = label;
                this.districtId = value;
                const res = await getTownByAreaId({countyId: value});
                console.log(res, '乡镇');
                this.ids = [];
                this.ids.push(this.provinceId);
                this.ids.push(this.cityId);
                this.ids.push(this.districtId);
                this.address = '';
                this.names = [];
                this.names.push(this.province);
                this.names.push(this.city);
                this.names.push(this.district);
                this.address = this.names[0] + this.names[1] + this.names[2];
                this.$emit('change', this.ids, this.names);
                if(JSON.stringify(res.data.result) == '{}'){
                    this.townList = {}
                    this.showDistrict= true;
                    this.showTown= false;

                    this.districtActive = true;
                    this.townActive = false;

                    this.showSelectAddress = false; //关闭挑选地址列表
                }else {
                    this.townList = res.data.result;
                    this.showDistrict= false;
                    this.showTown= true;

                    this.districtActive = false;
                    this.townActive = true;
                }

                this.showProvinces = false;
                this.showcitys = false;

                this.provinceActive = false;
                this.cityActive = false;

                this.town = '' //清空乡镇
            },
            selectTown(value, label){
                this.town = label;
                this.townId = value;

                this.ids = [];
                this.ids.push(this.provinceId);
                this.ids.push(this.cityId);
                this.ids.push(this.districtId);
                this.ids.push(this.townId);
                this.address = '';
                this.names = [];
                this.names.push(this.province);
                this.names.push(this.city);
                this.names.push(this.district);
                this.names.push(this.town);
                this.address = this.names[0] + this.names[1] + this.names[2] + this.names[3];
                this.$emit('change', this.ids, this.names);
                this.showSelectAddress = false; //关闭挑选地址列表
            },
            toggleCity(){
                this.showProvinces = false;
                this.showcitys = true;
                this.showDistrict=false;
                this.showTown=false;

                this.provinceActive = false;
                this.cityActive = true;
                this.districtActive = false;
                this.townActive = false;
            },
            toggleDistrice(){
                this.showProvinces = false;
                this.showcitys = false;
                this.showDistrict= true;
                this.showTown=false;

                this.provinceActive = false;
                this.cityActive = false;
                this.districtActive = true;
                this.townActive = false;
            },
            toggleTown(){
                this.showProvinces = false;
                this.showcitys = false;
                this.showDistrict= false;
                this.showTown=true;

                this.provinceActive = false;
                this.cityActive = false;
                this.districtActive = false;
                this.townActive = true;
            },
            closeAddress(){
                this.showSelectAddress = false; //关闭挑选地址列表
            },
            async initAddress(newValue){   //初始化省市区镇
                if(newValue.length != 0){
                    this.address = '';
                    this.provinceId = newValue[0];
                    this.cityId = newValue[1];
                    this.districtId = newValue[2];
                    this.townId = newValue[3] ? newValue[3] : '';
                    const resArr = await Promise.all([getAllProvince(),getCityByProvinceId({provinceId: this.provinceId}), getAreaByCityId({cityId: this.cityId}), getTownByAreaId({countyId: this.districtId})]);
                    resArr.forEach((item, index) => {
                        if(index == 0){
                            for(let k in resArr[0].data.result){
                                if(this.provinceId == resArr[0].data.result[k]){
                                    this.province = k;
                                    this.address += k;
                                }
                            }
                        }
                        if(index == 1){
                            this.citysList = resArr[1].data.result;
                            for(let k in resArr[1].data.result){
                                if(this.cityId == resArr[1].data.result[k]){
                                    this.city = k;
                                    this.address += k;
                                }
                            }
                        }
                        if(index == 2){
                            this.districtList = resArr[2].data.result;
                            for(let k in resArr[2].data.result){
                                if(this.districtId == resArr[2].data.result[k]){
                                    this.district = k;
                                    this.address += k;
                                }
                            }
                        }
                        if(index == 3){
                            this.townList = resArr[3].data.result;
                            for(let k in resArr[3].data.result){
                                if(this.townId == resArr[3].data.result[k]){
                                    this.town = k;
                                    this.address += k;
                                }
                            }
                        }
                    });
                }
            },
            selectAddress(){
                this.showSelectAddress = !this.showSelectAddress;
            }
        },
        watch: {
            async addressCode(newValue, oldValue){  
                this.addressCode = newValue;
                this.initAddress(newValue); //通过监听地址的变化来初始化选中的地区
            },
            showSelectAddress(newValue, oldValue){
                if(newValue){
                    this.ids.length != 0 ? this.initAddress(this.ids) : this.initAddress(this.addressCode);   //通过监听地址的变化来初始化选中的地区
                }
            }
        }
    }
</script>
<style lang="scss">
    .address-select-wrapper{
        width: 100%;
        background-color: #fff;
        .w-addres-mask{
            z-index: 9999;
            position: fixed;
            top:0;
            bottom: 0;
            left: 0;
            right: 0;
            background-color: rgba(0,0,0,.6);
            z-index: 1;
        }
        .w-address-bg{
            // z-index: 9999;
            // position: fixed;
            // top:0;
            // bottom: 0;
            // left: 0;
            // right: 0;
            // background-color: rgba(0,0,0,.6);
            .w-address-content{
                width: 100%;
                height: 700px;
                position: absolute;
                bottom: -700px;
                left: 0;
                background-color: #fff;
                transform: translate3d(0, -100%, 0);
                z-index: 2;
                &.fold-enter-active, &.fold-leave-active{
                    transition: all 0.5s
                }
                &.fold-enter, &.fold-leave-active{
                    transform: translate3d(0, 0, 0)
                }
                .content-header{
                    position: relative;
                    text-align: center;
                    color: #666;
                    line-height: 80px;
                    .close-w-address{
                        position: absolute;
                        width: 40px;
                        height: 40px;
                        top: 50%;
                        right: 80px;
                        transform: translateY(-50%);
                        /*关闭按钮*/
                        .close {
                            width: 40px;
                            height: 40px;
                            position: relative;
                            display: block;
                        }
                
                        .close:before, .close:after {
                            content: '';
                            position: absolute;
                            top: 50%;
                            width: 40px;
                            height: 1px;
                            background-color: #888;
                            -webkit-transform: rotate(45deg);
                            transform: rotate(45deg);
                        }
                
                        .close:after {
                            -webkit-transform: rotate(-45deg);
                            transform: rotate(-45deg);
                        }
                    }
                }
                .content-body{
                    .active-item{
                        color: #1086ff!important;
                    }
                    .selected-list{
                        border-top: 1px solid #ccc;
                        border-bottom: 1px solid #ccc;
                        line-height: 70px;
                        display: flex;
                        .selected-list-item{
                            margin: 0 40px;
                            color: #333;
                            font-weight: bold;
                            font-size: 30px;
                        }
                    }
                    .w-area{
                        overflow-y: scroll;
                        height: 550px;
                        .w-area-list{
                            .area-list-item{
                                list-style: none;
                                color: #333;
                                margin: 20px 0 30px 40px;
                                font-weight: bold;
                                font-size: 30px;
                            } 
                        }
                    }
                }
            }
        }
    }
</style>

