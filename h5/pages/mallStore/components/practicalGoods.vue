<!-- 实物商品 -->
<template>
    <view class="practicalGoods">
          <image v-if="shoppingMallVOList && shoppingMallVOList.length > 0" src="../../../static/image/pointsMall/icon1.png"  class="pic"/>
        <view class="listFather">
            <view class="box" v-for="(item,index) in shoppingMallVOList" :key='index'>
                <image class="img" :src="$config.getImgUrl(item.imgUrlApp)" />
                <view class="title com">{{item.name}}</view>
                <view class="price com">{{transform(item.amount)}}{{ currency }}</view>
                <view class="btn" @click="changeProduce(item,index)">{{ $t('立即兑换') }}</view>
            </view>
            <view class="box lastBox" >
                <view class="fbox">
                    <view>{{ $t('更多惊喜') }}</view>
                    <view>{{ $t('敬请期待') }}</view>
                </view>

            </view>
        </view>
    </view>
</template>
<script>
import mailStore from '../store'
export default {
    props: ["currency"],
    data(){
        return{
            window: window,
            shoppingMallVOList:[],
            limitCount:0   //控制加减的数量
        }
    },
    mounted(){
    },
    methods:{
        // 正则加逗号
        transform(num) {
            return num.toString().replace(/\B(?=(\d{3})+$)/g, ",")
        },
        changeProduce(item,index){
          mailStore.commit("setChangeItem", item);
            // this.$router.push({
            //     path:'/exchangeGoods',
            //     query:{
            //         type:2,
            //         isProInfo:true,
            //         index:index,
            //         limitCount:this.limitCount
            //     }
            // })
            const url = `/pages/mallStore/exchangeGoods?type=2&isProInfo=true&index=${index}&limitCount=${this.limitCount}`
            uni.navigateTo({
                url: url
            })
        }
    }
}
</script>
<style lang="scss" scoped>
    .practicalGoods{
        .pic{
            position: relative;
            left: 50%;
            transform: translateX(-50%);
            margin: 20px auto 20px;
            width: 124px;
            height: 42px;
        }
        .listFather{
            padding: 0 5px;
            color: #fff;
            display: flex;
            justify-content: space-between;
            flex-wrap: wrap;
            .box{
                width: 49%;
                height: 226px;
                padding-top: 12px;
                box-sizing: border-box;
                background: url('../../../static/image/pointsMall/goodBg.png') no-repeat;
                background-size: 100% 100%;
                border-radius: 8px;
                margin-bottom: 2%;
                text-align: center;
                color: #333;
                .img{
                    width: 104px;
                    height: 104px;
                    margin: 0 auto;
                    display: block;
                }
                .com{
                    font-size: 12px;
                    text-overflow: ellipsis;
                    white-space: nowrap;
                    overflow: hidden;
                }
                .title{
                  color: #333;
                  line-height: 30px;
                  font-size: 14px;
                  font-style: normal;
                  font-weight: 600;
                  padding: 0 13px 0 13px;
                }
                .price{
                    color: #db510a;
                    font-weight: 600;
                }
                .btn{
                    width: 110px;
                    height: 26px;
                    display: block;
                    margin: 15px auto 0;
                    box-shadow: inset 0 0 0 1px #f9e9c5;
                    line-height: 26px;
                    color: #FFF;
                    border-radius: 40px;
                    background: linear-gradient(180deg, #FCD78D 0%, #CCA456 100%);
                }
            }
            .lastBox{
                display: flex;
                justify-content: center;
                align-items: center;
                p{
                    margin: 0;
                    color: #616886;
                }
            }
        }

    }
</style>
