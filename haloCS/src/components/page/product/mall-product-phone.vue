<template>
    <div class="container ">
        <v-header></v-header>

        <div class="clearfix pageContain">
            <div class="contain_head clearfix">
                <div class="contain_head_left">
                    <v-photo :imgurl="imgList"></v-photo>
                </div>
                <div class="contain_head_right">
                    <h1>{{common.name}}</h1>
                    <p class="right_slogan">{{common.title}}</p>
                    <div class="right_price">
                        <a>价&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;格：</a>
                        <span>￥{{form.price.toFixed(2)}}</span>
                    </div>
                    <div class="right_selecct">
                        <dl v-if="common.version !== undefined">
                            <dt class="right_selecct_rom_lab">版&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本：</dt>
                            <dd v-for="item in common.version" class="right_selecct_rom_lab">
                                <el-button size="small" @click="goProduct(item.id)" v-if="item.type"
                                           :class="{selected:common.name == item.type}">
                                    {{item.type}}
                                </el-button>
                                <el-button size="small" @click="goProduct(item.id)" v-else
                                           :class="{selected:common.name == item.type}">
                                    {{item}}
                                </el-button>
                            </dd>
                        </dl>
                        <dl>
                            <dt class="right_selecct_rom_lab">网络类型：</dt>
                            <dd v-for="item in common.nettype" class="right_selecct_rom_lab"
                                @click="form.nettype = item">
                                <el-button :class="{selected:form.nettype == item}">{{item}}</el-button>
                            </dd>
                        </dl>
                        <dl>
                            <dt class="right_selecct_item_lab">颜色分类：</dt>
                            <dd v-for="(item,index) in filterColor" class="right_selecct_item right_selecct_item_lab">
                                <a @click="changeColor(item.name,index);form.imgUrl = item.img"
                                   :class="{selected:form.color == item.name}">
                                    <img v-lazy="item.img" width="32px" :key="item.img"><span>{{item.name}}</span>
                                </a>
                            </dd>
                        </dl>
                        <dl>
                            <dt class="right_selecct_rom_lab">内存容量：</dt>
                            <dd v-for="(item,index) in common.rom" class=" right_selecct_rom_lab"
                                @click="form.rom = item.size;form.price = item.price">
                                <el-button :class="{selected:form.rom == item.size}">{{item.size}}</el-button>
                            </dd>
                        </dl>
                    </div>
                    <v-suport :brand="common.name"></v-suport>
                    <div class="right_count">
                        <dl>
                            <dt class="right_selecct_item_lab">数&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;量：</dt>
                            <dd class="right_selecct_item_lab">
                                <el-input-number v-model="form.buyCount" :min="1" :max="10" size="mini"></el-input-number>
                            </dd>
                        </dl>

                        <div class="right_button">
                            <el-button type="danger" size="medium" class="right_buynow" @click="buyNow">
                                立即购买
                            </el-button>
                            <el-button type="primary" size="medium" class="right_buynow" @click="addCart">
                                加入购物车
                            </el-button>
                        </div>
                    </div>
                </div>
            </div>

            <v-detail :detailData="detailData"></v-detail>

            <v-footer></v-footer>

        </div>
        <v-hover :name="common.name" :nettype="form.nettype" :buyCount="form.buyCount"
                 :price="form.price"
                 :rom="form.rom" :colorName="form.color" @buyNow="buyNow"></v-hover>
        <el-dialog
                :visible.sync="centerDialogVisible"
                width="30%"
                @open="setTimeClose">
            <div class="cartDialog" v-if="form.color != ''">
                <i class="el-icon-circle-check-outline" style="color: #31a5e7"></i>
                <span style="color: #31a5e7">已成功加入购物车</span>
                <p @click="goRouter('mallcart')" style="cursor: pointer">去购物车结算 ></p>
            </div>
            <div v-else>
                <p style="color: #cc0000">请选择商品颜色!</p>
            </div>
        </el-dialog>
    </div>
</template>
<script>
    import vHeader from '../../common/header/page/header';
    import vPhoto from './common/photoShow';
    import bus from '../../common/bus.js';
    import vSuport from './common/suport';
    import vDetail from './common/detail';
    import vFooter from '../../common/footer';
    import vHover from './common/hoverBar';
    import qs from 'qs';
    export default {
        data() {
            return {
                common: [],
                form: {
                    color: '',
                    price: 0,
                    buyCount: 1,
                    nettype: '',
                    rom: '',
                    imgUrl: ""
                },
                sumPrice: 0,
                selectColor: 0,
                centerDialogVisible: false,
                detailData:[],
                source:'',
                imgList:[]
            }
        },
        components: {
            vHeader, vPhoto, vSuport, vDetail, vFooter, vHover
        },
        methods: {
            buyNow(){
                const expireTime = sessionStorage.getItem('expireTime');
                const nowTime = new Date().getTime();
                if(expireTime && nowTime < expireTime){
                    let buyForm = {
                        "proId": this.$route.query.proId,
                        "imgUrl": this.form.imgUrl,
                        "title": this.common.name + " " + this.form.nettype + " " + this.form.color + " " + this.form.rom,
                        "price": this.form.price,
                        "total": this.form.price * this.form.buyCount,
                        "number": this.form.buyCount
                    }
                    var url = this.$rootUrl + "/api/order/buyNow";
                    const options = {
                        method: 'POST',
                        url: url,
                        data: buyForm
                    };
                    this.$axios(options).then((res) => {
                        let item = res.data.data;
                        if (item.data) {
                            if (item.errorCode == 0) {
                                this.$store.commit("ORDER",{
                                    id:item.data.orderId,
                                    address:item.data.address,
                                    product:item.data.orderProduct
                                });
                                this.$router.push({path: "/mallCheck", query: {type: 1}})
                            }else{
                                throw item.msg;
                            }
                        }
                    }).catch(errorMsg => {
                        this.$message.error(errorMsg);
                    })
                }
                else {
                    sessionStorage.setItem('pageHistory',this.$route.fullPath);
                    this.$router.push({path: "/login"});
                }
            },
            addCart() {
                let buyForm = {
                    "proId": this.$route.query.proId,
                    "imgUrl": this.form.imgUrl,
                    "title": this.common.name + " " + this.form.nettype + " " + this.form.color + " " + this.form.rom,
                    "price": this.form.price,
                    "number": this.form.buyCount
                }
                var url = this.$rootUrl + "/api/carts/addCart";
                const options = {
                    method: 'POST',
                    url: url,
                    data: buyForm
                };
                this.$axios(options).then((res) => {
                    let item = res.data.data;
                    if (item.data) {
                        if (item.errorCode == 0) {
                            this.centerDialogVisible = true;
                            if (item.cookie) {
                                document.cookie = 'cart=' + item.cookie;
                            }
                            bus.$emit("cart", 1);
                        }else{
                            throw item.msg;
                        }
                    }
                }).catch(errorMsg => {
                    this.$message.error(errorMsg);
                });

            },
            changeColor(name, index) {
                this.form.color = name;
                this.selectColor = index;
                bus.$emit("pic", this.selectColor);
            },
            goRouter(that) {
                this.$router.push({path: "/" + that});
            },
            goProduct(id) {
                if (id <= 10)
                    this.$router.push({path: "/mallProductPhone", query: {proId: id}})
                else
                    this.$router.push({path: "/mallProductOther", query: {proId: id}})
            },
            getData() {
                var CancelToken = this.$axios.CancelToken
                this.source = CancelToken.source()
                this.cancelRequest();

                var proId = this.$route.query.proId;
                var url = this.$rootUrl + "/api/product/productDetail";

                const options = {
                    method: 'POST',
                    url: url,
                    data: {
                        proId: proId
                    },
                    cancelToken: new this.$axios.CancelToken((c) => {
                        this.source = c;
                    })
                };

                this.$axios(options).then((res) => {
                    let item = res.data.data;
                    if (item.errorCode == 0 && item.data.itemDetail.specificationJson) {
                        this.common = JSON.parse(item.data.itemDetail.specificationJson);
                        this.form.rom = this.common.rom[0].size;
                        this.form.price = this.common.rom[0].price;
                        this.form.nettype = this.common.nettype[0];
                        this.detailData = item.data.itemDetail.detailImg;
                        this.imgList = item.data.imgurl;
                    } else if (item.errorCode != 0) {
                        throw item.msg;
                    } else if (!item.data.itemDetail.specificationJson) {
                        throw '产品信息加载错误';
                        this.$router.go(-1);
                    }
                }).catch(errorMsg => {
                    this.$message.error(errorMsg);
                });
            },
            cancelRequest(){
                if(typeof this.source ==='function'){
                    this.source('终止请求')
                }
            }
        },
        computed: {
            filterColor() {
                let color = []
                const item = this.common.color;
                if (item) {
                    for (let i = 0; i < item.length; i++) {
                        let index = item[i].lastIndexOf(":")
                        color.push({name: item[i].substr(index + 1), img: item[i].slice(0, index)})
                    }
                    this.form.color = color[0].name;
                    this.form.imgUrl = color[0].img;
                }
                return color;
            }
        },
        created() {
            this.getData();
            window.scroll(0, 0);
        },
        watch: {
            '$route'(to, from) {
                this.getData();
            }
        }

    }
</script>
<style lang="less">
    .mall-product {
        /*position: absolute;*/
        /*top: 0px;*/
        /*left: 0px;*/
    }

    .contain_head {
        width: 100%;
        position: relative;
    }

    .contain_head_left{
        position: relative;
        width: 40%;
        height: 422px;
        float: left;
    }
    .contain_head_right {
        position: relative;
        width: 50%;
        float: left;
        padding-left: 1%;
        box-sizing: content-box;
    }

    .contain_head_right h1 {
        font-size: 24px;
        font-weight: 400;
    }

    .right_slogan {
        color: #e22841;
        font-size: 16px;
        margin-top: 10px;
    }

    .right_price {
        width: 700px;
        height: 50px;
        background: #F0F0F0;
        margin-top: 20px;
    }

    .right_price a {
        position: relative;
        top: 7px;
        left: 10px;
        color: #707070;
        font-size: 12px;
    }

    .right_price span {
        color: #e22841;
        font-size: 24px;
        position: relative;
        top: 11px;
        left: 15px;
    }

    .right_selecct {
        margin-top: 30px;
        border-bottom: 1px solid #dcdcdc;
        padding-bottom: 10px;
    }

    .right_selecct dl, .right_count dl {
        font-size: 12px;
        color: #666666;
        position: relative;
        left: 10px;
        margin-bottom: 10px;
    }

    .right_selecct dt {
        position: relative;
        top: 10px;
        vertical-align: top;
    }

    .right_selecct dd, .right_count dd {
        margin-left: 15px;
    }

    .right_selecct_item {
        display: inline;
    }

    .right_selecct_item a {
        width: 100px;
        height: 39px;
        display: inline-block;
        border: 1px solid #dcdcdc;
        border-radius: 5px;
        margin-bottom: 10px;
        text-align: center;
        cursor: pointer;
    }

    .selected {
        border: 1px solid #31a5e7 !important;
    }

    .right_selecct_item a img {
        margin-left: 10px;
        margin-top: 2px;
    }

    .right_selecct_item a span {
        display: inline-block;
        width: 55px;
        position: relative;
        left: 6px;
        top: -10px;
        color: #666666;
    }

    .right_selecct_item_lab, .right_selecct_rom_lab {
        display: inline-block;
        margin-bottom: 15px;
    }

    .right_selecct_rom {
        width: 90px;
        height: 35px;
        border: 1px solid #31a5e7;
        border-radius: 5px;
        margin-right: 10px;
    }

    .right_selecct_rom span, .right_selecct_nettype span {
        position: relative;
        top: 8px;
        left: 12px;
        color: #666666;
    }

    .right_selecct_nettype {
        width: 100px;
        height: 32px;
        border: 1px solid #31a5e7;
        border-radius: 5px;
    }

    .right_support {
        margin-top: 15px;
        margin-bottom: 15px;
        border-bottom: 1px solid #dcdcdc;
    }

    .right_support dd {
        color: #666;
        position: relative;
        top: -16px;
        left: 80px;
        font-size: 12px;
    }

    .right_support dt {
        font-size: 12px;
        color: #666;
        position: relative;
        left: 11px;
    }

    .support-dd {
        display: inline;
        margin-right: 10px;
    }

    .support-dd span {
        position: relative;
        top: 0px;
        left: 2px;
    }

    .right_buynow {
        width: 120px;
        margin-left: 50px;
    }

    .right_button {
        margin-left: -40px;
        margin-top: 20px;
        margin-bottom: 20px;
    }

    .selectedSection {
        border-bottom: 2px solid #31a5e7;
        color: #31a5e7;
    }

    .cartDialog {
        position: absolute;
        left: 50%;
        top: 50%;
        transform: translate(-50%, -50%);
        font-size: 14px;
    }
    .cartDialog p {
        margin-top: 10px;
        margin-left: 18px;
        color: #31a5e7;
        cursor: pointer;
    }
</style>
