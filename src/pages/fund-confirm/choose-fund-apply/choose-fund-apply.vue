<template>
    <view class="batchPages" style="position: relative;">
        <qm-navbar title="付款申请列表" btnName='确定' @myEvent='onHeaderConfirm'></qm-navbar>
        <qm-filters ref="filter" :array="array" @changeLeft="onChangeLeft">
            <view slot="drawer">
                <view class="drawer-item">
                    <view class="item-lable">
                        申请日期(起)
                    </view>
                    <picker mode="date" class="item-input" @change="bindStartDateChange">
                        <view class="uni-input">{{formData.applyDateStart | dateFormate}}</view>
                    </picker>
                </view>
                <view class="drawer-item">
                    <view class="item-lable">
                        申请日期(止)
                    </view>
                    <picker mode="date" class="item-input" @change="bindEndDateChange">
                        <view class="uni-input">{{ formData.applyDateEnd | dateFormate}}</view>
                    </picker>
                </view>
                <view class="drawer-item">
                    <view class="item-lable">
                        单据号
                    </view>
                    <view class="item-input">
                        <input type="text" value="" v-model="formData.applyNo" style="font-size: 22upx; height: 52upx; line-height: 52upx" />
                    </view>
                </view>

                <view class="drawer-item">
                    <view class="item-lable">
                        客户
                    </view>
                    <view class="item-input">
                        <picker mode="selector" @change="bindPickerChange" :value="index" :range-key="'name'" :range="customerList">
                            <view>
                                {{formData.customerNo}}
                                <uni-icons type="arrowdown" color="#999" class="icon"></uni-icons>
                            </view>
                        </picker>
                    </view>
                </view>
                <view class="drawer-item">
                    <view class="item-lable">
                        贸易类型
                    </view>
                    <view class="item-input">
                        <picker mode="selector" @change="bindPickerChange2" :value="index2" :range-key="'label'" :range="tradeTypeList">
                            <view>{{formData.tradeType}}
                                <uni-icons type="arrowdown" color="#999" class="icon"></uni-icons>
                            </view>
                        </picker>
                    </view>
                </view>
                <view class="drawer-item">
                    <view class="item-lable">
                        交货市场
                    </view>
                    <view class="item-input">
                        <picker mode="selector" @change="bindPickerChange3" :value="index3" :range-key="'name'" :range="deliveryMarketCodeList">
                            <view>{{formData.deliveryMarketCode}}
                                <uni-icons type="arrowdown" color="#999" class="icon"></uni-icons>
                            </view>

                        </picker>
                    </view>
                </view>
                <view class="drawer-item">
                    <view class="item-lable">
                        业务经理
                    </view>
                    <view class="item-input">
                        <picker mode="selector" @change="bindPickerChange4" :value="index4" :range-key="'name'" :range="bizEmployeeList">
                            <view>{{formData.bizEmployeeCode}}
                                <uni-icons type="arrowdown" color="#999" class="icon"></uni-icons>
                            </view>
                        </picker>
                    </view>
                </view>
                <view class="uni-btn-v">
                    <button type="default" size="mini" class="reset" @click="onReset">重置</button>
                    <button type="primary" size="mini" class="confirm" @click="onConfirm">确定</button>
                </view>
            </view>
        </qm-filters>
        <scroll-view scroll-y="true" class="priceBatchcon" lower-threshold='50' @scrolltolower="lower">
            <view class="pricelist">
                <view class="priceBatch-item" v-for="(item,i) in list" :key='i'>
                    <view class="confirm-header">
                        <view class="confirm-header-badge">
                            <view class="tag">
                                {{item.confirmStatus | dataDictFormat('confirmStatus') }}
                            </view>
                        </view>
                        <view class="confirm-header-title">
                            {{ item.applyNo }}
                        </view>
                        <view class="confirm-header-time">
                            {{item.applyDate | dateFormate}}
                        </view>
                    </view>
                    <view class="confirm-content">
                        <view>{{ item.customerName }}</view>
                        <view>
                            申请单据类型：{{item.applyBillType | dataDictFormat('fundApplyBillType') }}
                        </view>
                        <view class="">
                            申请金额(币种)：{{ item.applyAmount}}({{item.currency | dataDictFormat('currency')}})
                        </view>
                        <view>
                            确认金额(币种)：{{ item.confirmAmount}}({{item.currency | dataDictFormat('currency')}})
                        </view>
                        <view>
                            最迟付款日期: {{ item.latestPaymentDate | dateFormate}}
                        </view>
                    </view>
                    <view class="confirm-header-bottom">
                        <button type="primary" size="mini" class="viewbtn" @click="detaile(item)">查看详情</button>
                        <button type="primary" v-bind:data-val="item.num" v-if="!item.disabled" size="mini" @click="onJoin(item)">立即加入</button>
                        <button type="primary" size="mini" v-if="item.disabled" style="opacity: 0.5;" @click="offJoin(item)">取消加入</button>
                    </view>
                </view>
                <view v-if="!hasMore" class="more">
                    ---------- 没有更多了 -----------
                </view>
            </view>
        </scroll-view>
        <view class="dialog" v-if='dialogVisible'>
            <view class="mask">
            </view>
            <view class="dialogCon" style="">
                <qm-dialog-list ref="uniPop" fileName='viewFundApplyInfo' :queryParams='popItem' @hideClick='hideClick'></qm-dialog-list>
            </view>
        </view>
    </view>
</template>

<script>
import request from '@/utils/request.js'
export default {
    data() {
        return {
            dialogVisible: false,
            array: ['全部'],
            hasMore: true,
            index: 0,
            index2: 0,
            index3: 0,
            index4: 0,
            customerList: [],
            bizEmployeeList: [],
            tradeTypeList: [],
            deliveryMarketCodeList: [],
            formData: {
                applyBillType: '',
                applyDateEnd: null,
                applyDateStart: null,
                applyNo: '',
                auditStatus: '3',
                bizDeptCode: '',
                bizEmployeeCode: '',
                bizOrganCode: '',
                confirmStatusNe: '2',
                customerNo: '',
                optEmployeeCode: '',
                qmQueryExtras: [],
            },
            ladingNoFormData: {
                ladingBillNo: '',
                customerNoList: [],
                qmQueryExtras: [],
                supplierCode: '',
            },
            productItemFormData: {
                customerNo: '',
            },
            popItem: {},
            list: [],
            type: '',
            current: 1,
        }
    },
    async onShow() {
        this.$store.state.dictData.confirmStatus.forEach((item) => {
            this.array.push(item.label)
        })
        this.tradeTypeList = this.$store.state.dictData.tradeType
        // 机构
        const customerList = await this.selectDataRequest({
            type: 'CUST_INFO',
            queryParams: {
                usingFlag: '1',
                auditStatus: '3',
            },
        })
        this.customerList = customerList.data
        // 机构
        const bizEmployeeList = await this.selectDataRequest({
            type: 'FUNC_EMPLOYEE',
        })
        // 交货市场
        const deliveryMarketCodeList = await this.selectDataRequest({
            type: 'DD_SPOT_MARKET',
        })
        this.deliveryMarketCodeList = deliveryMarketCodeList.data
        this.bizEmployeeList = bizEmployeeList.data
    },
    onLoad() {
		this.formData = Object.assign({}, this.formData, this.$Route.query)
        this.getList()
    },
    methods: {
        bindStartDateChange(e) {
            this.formData.applyDateStart = this.dateFormate(e.detail.value)
        },
        bindEndDateChange(e) {
            this.formData.applyDateEnd = this.dateFormate(e.detail.value)
        },
        dateFormate(date) {
            if (date) {
                const year = new Date(date).getFullYear() + ''
                const month =
                    new Date(date).getMonth() + 1 > 9 ?
                        new Date(date).getMonth() + 1 :
                        '0' + (new Date(date).getMonth() + 1)
                const day =
                    new Date(date).getDate() > 9 ?
                        new Date(date).getDate() :
                        '0' + new Date(date).getDate()
                return year + month + day
            }
        },
        selectDataRequest(params) {
            return request.post('/vrm/api/dd/selectData/list', {
                funcModule: '付款申请新增',
                funcOperation: '查询',
                data: params,
            })
        },
        bindPickerChange(e) {
            this.index = e.target.value
            if (this.customerList[e.target.value]) {
                this.formData.customerNo = this.customerList[e.target.value].code
            } else {
                this.formData.customerNo = ''
                this.index = 0
            }
        },
        bindPickerChange2(e) {
            this.index2 = e.target.value
            if (this.tradeTypeList[e.target.value]) {
                this.formData.tradeType = this.tradeTypeList[e.target.value].value
            } else {
                this.formData.tradeType = ''
            }
        },
        bindPickerChange3(e) {
            this.index3 = e.target.value
            if (this.deliveryMarketCodeList[e.target.value]) {
                this.formData.deliveryMarketCode = this.deliveryMarketCodeList[
                    e.target.value
                ].code
            } else {
                this.formData.deliveryMarketCode = ''
            }
        },
        bindPickerChange4(e) {
            this.index4 = e.target.value
            if (this.bizEmployeeList[e.target.value]) {
                this.formData.bizEmployeeCode = this.bizEmployeeList[
                    e.target.value
                ].code
            } else {
                this.formData.bizEmployeeCode = ''
            }
        },
        async getList() {
            const res = await request.post('/vrm/api/fund/viewFundApplyInfo/list', {
                defaultSortString: 'applyDate.asc, applyNo.asc',
                funcModule: '付款确认新增',
                funcOperation: '查询',
                isPage: true,
                size: 20,
                data: this.formData,
                current: this.current,
            })
            this.list = res.data
            if (this.current === 1) {
                this.list = res.data
            } else {
                this.list = [...this.list, ...res.data]
            }
            if (res.data.length < 20) {
                this.hasMore = !this.hasMore
                return
            }
        },
        onChangeLeft(val) {
            this.formData.contractType = val === 1 ? '1' : val === 2 ? '2' : ''
            this.current = 1
            this.getList()
        },
        lower() {
            if (this.hasMore) {
                this.current += 1
                this.getList()
            }
        },
        onConfirm() {
            this.$refs.filter.closeDrawer()
            this.current = 1
            this.getList()
        },
        onReset() {
            this.formData.applyDateStart = ''
            this.formData.applyDateEnd = ''
            this.formData.billType = ''
            this.formData.applyNo = ''
            this.formData.lotNo = ''
            this.formData.tradeType = ''
            this.formData.customerNo = ''
            this.formData.bizEmployeeCode = ''
        },
        onHeaderConfirm() {
            const list = this.list.filter((item) => item.disabled)
            if (list.length === 0) {
                uni.showToast({
                    title: '请选择一条信息',
                    icon: 'none',
                })
            } else {
                uni.showModal({
                    title: '提示',
                    content: '请确认当前数据已经准确无误，是否确定保存？',
                    success: (res) => {
                        if (res.confirm) {
                            let pages = getCurrentPages()
                            let nowPage = pages[pages.length - 1]
                            let prevPage = pages[pages.length - 2]
                            prevPage.$vm.selectFundApplyCallback(list[0])
                            this.$Router.back(1)
                        } else if (res.cancel) {
                            console.log('用户点击取消')
                        }
                    },
                })
            }
        },
        hideClick(va) {
            this.dialogVisible = false
        },
        detaile(item) {
            this.dialogVisible = true
            this.popItem = item
            // uni.redirectTo({
            // 	url: `/pages/fund-apply/choose-contract-info/contract-detail-list/contract-detail-list?queryParams=${JSON.stringify(item)}`
            // })
        },
        onJoin(item) {
            this.list.map((item) => {
                this.$set(item, 'disabled', false)
                this.$set(item, 'num', 0)
            })
            this.$set(item, 'disabled', !item.disabled)
            this.$set(item, 'num', 1)
        },
        offJoin(item) {
            this.$set(item, 'disabled', !item.disabled)
            this.$set(item, 'num', 1)
        },
    },
}
</script>

<style scoped lang="scss">
.opciton {
  opacity: 0.5;
}

.batchPages {
  background: #f5f5f5;

  .priceBatchcon {
    height: calc(100vh - 170upx);
    overflow: scroll;
    background-color: #f5f5f5;
  }

  .more {
    font-size: 28upx;
    text-align: center;
    color: #999;
    margin: 20upx 0;
  }

  .pricelist {
    font-size: 28upx;

    .priceBatch-item {
      background-color: #fff;
      margin: 30upx;
      padding: 20upx;
    }

    .confirm-header {
      width: 100%;
      height: 60upx;
      line-height: 60upx;
      display: flex;

      .confirm-header-badge {
        width: 180upx;

        .tag {
          font-size: 20upx;
          padding: 4upx 16upx;
          border-radius: 50upx;
          color: #fff;
          display: inline-block;
          line-height: 30upx;
          color: rgb(0, 143, 255);
          background: rgb(212, 233, 252);
        }
      }

      .confirm-header-title {
        width: 350upx;
        overflow: hidden;
        white-space: nowrap;
        text-overflow: ellipsis;
      }

      .confirm-header-time {
        width: 150upx;
        color: #999;
        font-size: 20upx;
        border-radius: 50upx;
        text-align: right;
      }
    }

    .confirm-content {
      color: #666;
      padding: 20upx;
      font-size: 24upx;
      line-height: 42upx;
    }

    .confirm-header-bottom {
      text-align: right;
      line-height: 50upx;

      uni-button {
        margin-left: 10upx;
        border-radius: 50upx;
      }

      .viewbtn {
        color: rgb(0, 143, 255);
        background: rgb(212, 233, 252);
      }
    }
  }
}

.drawer-item {
  margin-bottom: 20upx;

  .item-lable {
    text-align: left;
  }

  .item-input {
    flex: 1;
    height: 52upx;
    line-height: 52upx;
    background-color: #f9f9f9;
    font-size: 24upx;
    border: 2upx solid #f5f5f5;
    text-align: left;
    padding-left: 20upx;
    position: relative;

    .icon {
      position: absolute;
      right: 10upx;
    }
  }

  .msg {
    display: flex;
    font-size: 24upx;
    height: 52upx;
    line-height: 50upx;

    .msg-select {
      flex: 1;
      font-size: 22upx;
      background: #f1f1f1;
      border: 2upx solid #d3d3d3;
      margin: 0 5upx;
      color: #666;
    }

    .msg-select-active {
      color: #2484cf;
      background: #b7defc;
      border: 2upx solid #2484cf;
    }
  }
}

.uni-btn-v {
  position: absolute;
  bottom: 10upx;
  left: 30upx;
  right: 30upx;

  uni-button {
    width: 50%;

    &::after {
      border-radius: 0;
    }
  }

  .reset {
    border-top-right-radius: 0;
    border-bottom-right-radius: 0;
  }

  .confirm {
    border-top-left-radius: 0;
    border-bottom-left-radius: 0;
  }
}

.dialog {
  .mask {
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.2);
    position: absolute;
    top: 0;
  }

  .dialogCon {
    background-color: #fff;
    width: 100%;
    height: 100%;
    position: absolute;
    top: 0;
  }
}
</style>
