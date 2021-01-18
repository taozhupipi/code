<template>
    <AForm :form="form">
        <Row>
            <ACol>
                <ACollapse accordion v-model="activeKey">
                    <template #expandIcon="props">
                        <a-icon type="caret-right" :rotate="props.isActive ? 90 : 90" />
                    </template>
                    <ACollapsePanel key="1" header="基础设置" :style="customStyle">
                        <template>
                            <!-- <div> -->
                            <div class="main">
                                <img
                                    :src="`/vortex/rest/cloud/np/file/downloadFile?id=` + fileId"
                                    style="width: 100%; height: 100%; display: none"
                                    ref="img"
                                />
                                <!-- :src="imgUrl" -->

                                <canvas ref="canvas" :width="cvWeight" :height="cvHeight" />
                            </div>
                            <div>
                                <AButton :disabled="!channelId" @click="() => capture(channelId)"
                                    >抓拍</AButton
                                ><br />
                                <AButton :disabled="!channelId" @click="() => getPicture()"
                                    >获取图片</AButton
                                ><br />
                                <AButton :disabled="!channelId" @click="clearDraw()"
                                    >清除区域</AButton
                                >
                                <AButton
                                    style="display: block"
                                    :disabled="!channelId"
                                    @click="save()"
                                    >保存区域</AButton
                                >
                                <AButton :disabled="!channelId" @click="addArea()"
                                    >新增区域</AButton
                                >
                                <!-- <AButton :disabled="!channelId" @click="set">初始化设置</AButton> -->
                                <a-radio-group
                                    style="display:block;width:100px;margin-left"
                                    @change="RadioOnChange"
                                    v-for="item in radioData"
                                    :key="item.id"
                                    v-model="radioValue"
                                    :defaultValue="1"
                                >
                                    <a-radio
                                        style="display: block; margin-top: 5px"
                                        :value="item.value"
                                        :name="item.name"
                                    >
                                        {{ item.value }}
                                    </a-radio>
                                </a-radio-group>
                            </div>
                            <!-- </div> -->
                        </template>
                    </ACollapsePanel>
                </ACollapse>
            </ACol>
            <ACol>
                <ACollapse accordion v-model="activeKey" style="margin-top: 20px">
                    <template #expandIcon="props">
                        <a-icon type="caret-right" :rotate="props.isActive ? 90 : 90" />
                    </template>
                    <ACollapsePanel key="1" header="推送设置" :style="customStyle">
                        <ACol :span="20" :offset="1">
                            <AFormItem label="推送间隔" class="vtx-form-item">
                                <div>
                                    <ASwitch
                                        v-model="publishConfig.realtime"
                                        checked-children="实时推送"
                                        un-checked-children="周期推送"
                                        default-checked
                                        @change="realtimeChange"
                                    />

                                    <div class="input-only" style="margin-left: 10px">
                                        <Input
                                            addon-before="间隔"
                                            placeholder="请输入"
                                            addon-after="ms"
                                            v-if="publishConfig.realtime === false"
                                            v-model="publishConfig.interval"
                                        />
                                    </div>
                                </div>
                            </AFormItem>
                        </ACol>
                        <ACol :span="20" :offset="1">
                            <AFormItem label="推送方式" class="vtx-form-item">
                                <ACol :span="18">
                                    <ACheckbox
                                        v-model="publishConfig.enableKafka"
                                        value="publishConfig.enableKafka"
                                    >
                                        JMS消息推送</ACheckbox
                                    >
                                </ACol>
                                <ACol :span="18">
                                    <ACheckbox
                                        style="margin-top: 10px"
                                        value="publishConfig.enableHttp"
                                        v-model="publishConfig.enableHttp"
                                        >API调用
                                    </ACheckbox>
                                    <div
                                        class="input-row"
                                        style="margin-bottom: 40px; margin-top: -45px"
                                        v-for="(deal, i) in publishConfig.httpUrlList"
                                        :key="i"
                                    >
                                        <div>
                                            <Input
                                                placeholder="请输入"
                                                v-model="publishConfig.httpUrlList[i]"
                                            />
                                        </div>

                                        <div
                                            style="
                                                display: flex;
                                                margin-left: 20px;
                                                margin-top: 3px;
                                            "
                                        >
                                            <AButton
                                                icon="plus"
                                                @click="addDealOption"
                                                v-if="i === publishConfig.httpUrlList.length - 1"
                                                shape="circle"
                                            />
                                            <AButton
                                                icon="close"
                                                @click="removeDealOption"
                                                v-if="
                                                    i !== 0 && publishConfig.httpUrlList.length > 1
                                                "
                                                shape="circle"
                                            />
                                        </div>
                                    </div>
                                </ACol>
                            </AFormItem>
                        </ACol>
                    </ACollapsePanel>
                </ACollapse>
            </ACol>
        </Row>
    </AForm>
</template>
<script>
import Vue from 'vue';
import cloneDeep from 'lodash/cloneDeep';
import merge from 'lodash/merge';
import { VtxUtil } from '@/utils/util';

import { processTreeManageService } from '@/services/processTreeManageService';

import {
    Row,
    Col as ACol,
    Form,
    Input,
    // Divider,
    Collapse as ACollapse,
    Icon as AIcon,
    Radio as ARadio,
    Button as AButton,
    Checkbox as ACheckbox,
    Select as ASelect,
    Switch as ASwitch,
    message,
} from 'ant-design-vue';
Vue.use(Form);
Vue.use(ACollapse);
Vue.use(ARadio);
Vue.use(ACheckbox);
Vue.use(ASelect);
// 模拟双击事件
let clickCount = 0;
const strokeStyle = 'red';
const lineWidth = 1;
const fillStyle = 'rgba(0,0,0,0.2)';
export default {
    data() {
        this.first = [];
        this.drawingType = 'add';
        this.areaPoint = [];
        this.defaultAreaPoint = [];
        return {
            fileId: '',
            channelId: '',
            cvHeight: 0,
            cvWeight: 560,
            visible: false,
            formData: {
                name: '',
            },
            interval: '',

            customStyle:
                'background: #f7f7f7;margin-bottom: 0px;border: 0;overflow: hidden,width:100%',
            activeKey: ['1'],
            value: 1,

            // 推送方式
            publishConfig: {
                realtime: false, //true实时推送，false 周期推送
                interval: '', //推送间隔
                enableKafka: false, // JMS
                enableHttp: false, //API调用
                httpUrlList: [''], //API调用的列表
            },
            areaMap: {
                additionalProp1: '',
            },
            //   单选框数据
            radioValue: '',
            radioId: 2,
            radioData: [
                {
                    id: 1,
                    value: '区域1',
                    name: '区域1',
                },
            ],
            areaPoints: [],
            radioObj: {},
        };
    },
    components: {
        Row,
        ACol,
        Input,
        ACollapse,
        AIcon,
        AButton,
        ACheckbox,
        ASwitch,
    },
    props: {
        config: {
            type: Object,
            default() {
                return {};
            },
        },
        otherParams: {
            type: Object,
            default() {
                return {};
            },
        },
    },
    watch: {
        config: {
            //深度监听，可监听到对象、数组的变化
            handler: function (val, oldVal) {
                const { publishConfig, areaMap } = cloneDeep(val);
                this.publishConfig = publishConfig;
                const areaPoint = (areaMap?.additionalProp1 || '')
                    .split(';')
                    .filter((item) => item)
                    .map((item) => item.split(',').map((num) => Number(num)));
                this.areaPoint = cloneDeep(areaPoint);
                this.defaultAreaPoint = cloneDeep(areaPoint);

                const _t = this;
                if (areaPoint.length > 0) {
                    this.$nextTick(() => {
                        setTimeout(() => {
                            _t.canvasInit();
                            _t.reDraw(areaPoint);
                            _t.$refs.canvas.style.cursor = 'default';
                        }, 600);
                    });
                }
            },
            deep: true,
        },
    },

    created() {
        this.form = this.$form.createForm(this, {
            name: 'LocalOption',
        });
        const channelId = VtxUtil.getUrlParam('channelId');
        this.channelId = channelId;
        this.getPicture(channelId);
    },
    methods: {
        RadioOnChange(e) {
            // console.log(e.target.value , this.areaPoints[1]);
            //获取单选框的下标值
            let radioIndex = e.target.value.slice(2) - 1;
            let xx = this.areaPoints;
            // let first=this.areaPoint.pop()
            // let ap=this.areaPoint.slice(-1)
            // console.log(first,'+++++++',ap);
            const t = this;
            let { cvWeight, $refs } = t;
            const { img } = $refs;
            if (t.ctx) {
                t.ctx.clearRect(0, 0, cvWeight, (cvWeight / img.width) * img.height);
                t.ctx.drawImage(img, 0, 0, cvWeight, (cvWeight / img.width) * img.height);
                // t.first = this.areaPoints.slice(-1);
                // t.areaPoint = this.areaPoints.pop();
                t.drawingType = 'add';
            }
        },
        show(formData = {}) {
            this.visible = true;
            this.$set(this, 'formData', formData);
            this.$nextTick(() => {
                this.form.setFieldsValue({
                    name: formData.name,
                });
            });
        },

        addDealOption() {
            this.publishConfig.httpUrlList.push('');
        },

        removeDealOption() {
            this.publishConfig.httpUrlList.pop();
        },

        // 推送间隔
        realtimeChange(realtime) {
            this.realtime = realtime;
            //   console.log(`切换 to ${realtime}`);
        },

        // 抓拍图片
        capture(channelId) {
            const newChannelId = channelId || this.channelId;
            if (newChannelId) {
                processTreeManageService.capturePic({ channelId: newChannelId }).then(({ res }) => {
                    // console.log(res);
                });
            }
        },
        // 获取图片
        // fileId
        getPicture(channelId) {
            const _t = this;
            const newChannelId = channelId || this.channelId;
            if (newChannelId) {
                // console.log(newChannelId);
                processTreeManageService.getPic({ channelld: newChannelId }).then((res) => {
                    // console.log(res);
                    _t.fileId = res.ret;
                });
            }
            // console.log(fileId);
        },
        // 新增区域
        addArea() {
            // this.clearDraw();
            this.radioData.push({
                id: +new Date(),
                value: `区域` + this.radioId++,
                name: Math.random().toString(),
            });

            // 根据新增区域按钮往areaMap中动态添加对象
            Object.keys(this.areaMap).forEach((key, item) => {
                let additionalProp = key.slice(0, -1);
                key = additionalProp + this.radioData.length.toString();
                for (let index = 0; index < this.radioData.length; index++) {
                    this.areaMap[key] = '';
                }
            });
        },
        // 保存图片区域
        save() {
            const { areaPoint, first, cvHeight, cvWeight } = this;
            // 遍历areaPoint获取点位除画布的宽高得到百分比重新赋值
            areaPoint.forEach((val, key) => {
                let valW = val[0] / cvWeight;
                let valH = val[1] / cvHeight;
                val.length = 0;
                val.push(valW, valH);
            });
            // 遍历first获取点位除画布的宽高得到百分比重新赋值
            let firstWFlot = first[0] / cvWeight;
            let firstHFlot = first[1] / cvHeight;
            first.length = 0;
            first.push(firstWFlot, firstHFlot);
            let points = areaPoint.concat([first]);
            let additionalProp = points.join(';');
            this.areaPoints.push(additionalProp);
            // 遍历对象获取到里面的值
            Object.keys(this.areaMap).forEach((key, item) => {
                // 截取区域1、区域2、区域3.。。。。
                let additionalProp = key.slice(0, -1);
                key = additionalProp + this.radioData.length.toString();
                console.log(key);
                this.areaPoints.forEach((k, i) => {
                    this.areaMap[key] = k;
                });
            });
            console.log(this.areaMap);
            // this.areaMap.additionalProp1 = additionalProp1;
        },

        // 保存
        getFormData() {
            const { publishConfig, areaMap, areaPoint, first } = this;
            let points = areaPoint.concat([first]);
            let additionalProp1 = points.join(';');
            console.log(areaMap);
            return {
                config: {
                    publishConfig,
                    areaMap: {
                        ...areaMap,
                    },
                },
            };
        },
        // 初始化设置
        set() {
            const t = this;
            t.canvasInit();
            t.reDraw(t.defaultAreaPoint);
            t.$refs.canvas.style.cursor = 'default';
            this.drawingType = 'edit';
        },
        reDraw(points) {
            const t = this;
            let { cvWeight, $refs } = t;
            const { img } = $refs;

            if (t.ctx) {
                t.ctx.clearRect(0, 0, cvWeight, (cvWeight / img.width) * img.height);
                t.ctx.drawImage(img, 0, 0, cvWeight, (cvWeight / img.width) * img.height);
                t.ctx.beginPath();
                t.ctx.save();

                for (let i = 0; i < points.length; i++) {
                    t.ctx[i == 0 ? 'moveTo' : 'lineTo'](points[i][0], points[i][1]);
                }
                t.ctx.closePath();
                t.ctx.fill();
                t.ctx.stroke();
            }
            t.drawGraph(points);
        },

        drawGraph(points) {
            if (this.ctx) {
                this.ctx.save();
                this.ctx.lineWidth = 2;
                this.ctx.strokeStyle = '#000';
                points.forEach((p) => {
                    this.ctx.beginPath();
                    this.ctx.arc(p[0], p[1], 10, 0, Math.PI * 2, false);
                    this.ctx.closePath();
                    this.ctx.stroke();
                });
                this.ctx.restore();
            }
        },

        // 初始化画板
        canvasInit() {
            if (this.ctx) {
                this.ctx.strokeStyle = strokeStyle;
                this.ctx.lineWidth = lineWidth;
                this.ctx.fillStyle = fillStyle;
            }
        },

        // 清除绘图
        clearDraw() {
            const t = this;
            let { cvWeight, $refs } = t;
            const { img } = $refs;
            if (t.ctx) {
                t.ctx.clearRect(0, 0, cvWeight, (cvWeight / img.width) * img.height);
                t.ctx.drawImage(img, 0, 0, cvWeight, (cvWeight / img.width) * img.height);
                t.first = [];
                t.areaPoint = [];
                t.drawingType = 'add';
            }
        },
        // 判断当前点击的点位是否在选中的控制点内 point： 点击的点位，circle：圆心位置
        isPointInCircle(point, circle) {
            let r = 10;
            let p = { x: point[0], y: point[1] };
            let dis = Math.sqrt(
                (p.x - circle[0]) * (p.x - circle[0]) + (p.y - circle[1]) * (p.y - circle[1]),
            );
            if (dis <= r) {
                return true;
            }
            return false;
        },

        init() {
            const t = this;
            const { cvWeight, $refs } = t;
            const { img, canvas } = $refs;
            t.ctx = canvas.getContext('2d');
            // 图片加载的时候
            img.onload = function () {
                // 获取图片的实际宽、高
                const { width, height } = img;
                // 设置画布的高度
                t.cvHeight = (cvWeight / width) * height;
                t.$nextTick(() => {
                    // 插入图片以及画布
                    t.ctx.drawImage(img, 0, 0, cvWeight, (cvWeight / width) * height);
                });
            };
            const mouseMove = (e) => {
                let x1 = e.layerX,
                    y1 = e.layerY;
                let points = cloneDeep(t.areaPoint);
                points[t.index] = [x1, y1];
                t.reDraw(points);
                t.areaPoint = points;
            };
            canvas.onclick = function (e) {
                clickCount++;
                let x = e.layerX,
                    y = e.layerY;
                t.dClickTimer = setTimeout(function () {
                    if (clickCount === 1) {
                        //单击事件
                        if (t.drawingType === 'add') {
                            //新增事件
                            t.canvasInit();
                            if (t.first.length === 0) {
                                let points = [[x, y]];
                                t.ctx.beginPath();
                                t.ctx.moveTo(x, y);
                                t.ctx.fill();
                                t.drawGraph(points);
                                t.ctx.closePath();
                                t.first = [x, y];
                                t.areaPoint = points;
                            } else {
                                let points = cloneDeep(t.areaPoint);
                                if (points.length > 19) {
                                    message.warn('绘制点数不能大于20');
                                    return false;
                                }
                                points.push([x, y]);
                                t.reDraw(points);
                                t.areaPoint = points;
                            }
                        } else {
                            //编辑拖动控制点
                            for (let i = 0; i < t.areaPoint.length; i++) {
                                let point = t.areaPoint[i];
                                let flag = t.isPointInCircle([x, y], point);
                                if (flag) {
                                    t.index = i;
                                    canvas.style.cursor = 'crosshair';
                                    canvas.addEventListener('mousemove', mouseMove);
                                }
                            }
                        }
                    }
                    if (clickCount === 2) {
                        //双击事件
                        if (t.drawingType === 'add') {
                            //新增
                            let points = cloneDeep(t.areaPoint);
                            t.reDraw(points);
                            t.drawingType = 'edit';
                        } else {
                            //编辑
                            let points = cloneDeep(t.areaPoint);
                            points[t.index] = [x, y];
                            t.reDraw(points);
                            canvas.style.cursor = 'default';
                            canvas.removeEventListener('mousemove', mouseMove);
                            t.areaPoint = points;
                        }
                    }
                    clearTimeout(t.dClickTimer);
                    clickCount = 0;
                }, 300);
            };
        },
    },
    mounted() {
        this.init();
    },
};
</script>
<style lang="less" scoped>
@import '@/assets/css/layout.less';
@import '@/assets/css/input.less';

@{deep} .ant-modal-body {
    padding: 0;
}
@{deep} .ant-form-item label {
    position: relative;
    margin-bottom: 10px;
}
.main {
    width: 560px;
    float: right;
    margin-right: 200px;
}
</style>
