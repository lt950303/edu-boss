<template>
  <div class="course-create">
    <div slot="header" class="clearfix">
      <span>{{ isEdit ? '课程编辑' : '课程创建'}}</span>
    </div>
    <div class="text item">
      <el-steps :active="activeStep" finish-status="success">
        <el-step
          v-for="(item, index) in steps"
          @click.native="activeStep = index"
          :key="item.title"
          :title="item.title">
        </el-step>
      </el-steps>
    </div>
    <el-form label-width="120px">
      <div v-show="activeStep === 0">
        <el-form-item label="课程名称">
          <el-input  v-model="course.courseName"></el-input>
        </el-form-item>
        <el-form-item label="课程简介">
          <el-input  v-model="course.brief"></el-input>
        </el-form-item>
        <el-form-item label="课程概述">
          <el-input
            style="margin-bottom: 10px"
            v-model="course.previewFirstField"
            type="textarea"
            placeholder="概述1"
          ></el-input>
          <el-input
            v-model="course.previewSecondField"
            type="textarea"
            placeholder="概述2"
          ></el-input>
        </el-form-item>
        <el-form-item label="讲师姓名">
          <el-input v-model="course.teacherDTO.teacherName"></el-input>
        </el-form-item>
        <el-form-item label="讲师简介">
          <el-input v-model="course.teacherDTO.description"></el-input>
        </el-form-item>
        <el-form-item label="课程排序">
          <el-input-number  v-model="course.sortNum" label="描述文字"></el-input-number>
        </el-form-item>
      </div>
      <div v-show="activeStep === 1">
        <el-form-item  label="课程封面">
          <!-- 将上传组件封装成一个组件 -->
          <!-- 自定义组件 v-modal 如何实现？？？ -->
          <!--
            1. 组件需要根据绑定的数据进行图片预览
            2. 组件需要把上传成功的图片地址同步到绑定的数据中
            v-model 的本质还是父子组件通信
              1. 它会给子组件传递一个名字叫 value 的数据（Props）
              2. 默认监听 input 事件，修改绑定的数据（自定义事件）
           -->
           <course-image v-model="course.courseListImg" />
          <!-- <el-upload
            class="avatar-uploader"
            action="https://jsonplaceholder.typicode.com/posts/"
            :show-file-list="false"
            :http-request="handleUpload"
            :before-upload="beforeAvatarUpload">
            <img v-if="course.courseImgUrl" :src="course.courseImgUrl" class="avatar">
            <i v-else class="el-icon-plus avatar-uploader-icon"></i>
          </el-upload> -->
        </el-form-item>
        <el-form-item  label="解锁封面">
          <course-image v-model="course.courseImgUrl" :limit="1" />
        </el-form-item>
      </div>
      <div v-show="activeStep === 2">
        <el-form-item label="售卖价格">
          <el-input v-model.number="course.discounts" type="number">
            <template slot="append">元</template>
          </el-input>
        </el-form-item>
        <el-form-item label="商品原价">
          <el-input  v-model.number="course.price" type="number">
            <template slot="append">元</template>
          </el-input>
        </el-form-item>
        <el-form-item label="销量">
          <el-input v-model.number="course.sales" type="number">
            <template slot="append">单</template>
          </el-input>
        </el-form-item>
        <el-form-item label="活动标签">
          <el-input v-model="course.discountsTag"></el-input>
        </el-form-item>
      </div>
      <div v-show="activeStep === 3">
        <el-form-item label="限时秒杀开关">
          <el-switch
            v-model="course.activityCourse"
            active-color="#13ce66"
            inactive-color="#ff4949"
          >
          </el-switch>
        </el-form-item>
        <template v-if="course.activityCourse">
          <el-form-item label="开始时间">
            <el-date-picker
              type="date"
              v-model="course.activityCourseDTO.beginTime"
              placeholder="选择日期时间"
              value-format="yyyy-MM-dd"
            />
          </el-form-item>
          <el-form-item label="结束时间">
            <el-date-picker
              type="date"
              v-model="course.activityCourseDTO.endTime"
              placeholder="选择日期时间"
              value-format="yyyy-MM-dd"
            />
          </el-form-item>
          <el-form-item label="秒杀价">
            <el-input v-model.number="course.activityCourseDTO.amount" type="number">
              <template slot="append">元</template>
            </el-input>
          </el-form-item>
          <el-form-item label="秒杀库存">
            <el-input v-model.number="course.activityCourseDTO.stock" type="number">
              <template slot="append">个</template>
            </el-input>
          </el-form-item>
        </template>
      </div>
      <div v-show="activeStep === 4">
        <el-form-item label="课程详情">
          <text-editor v-model="course.courseDescriptionMarkDown" />
            <!-- <el-input v-model="course.courseDescriptionMarkDown" type="textarea">
            </el-input> -->
        </el-form-item>
        <el-form-item label="是否发布">
          <el-switch
            v-model="course.status"
            :active-value="1"
            :inactive-value="0"
            active-color="#13ce66"
            inactive-color="#ff4949"
          />
        </el-form-item>
        <el-button
          style="margin-top: 12px;"
          type="primary"
          @click="handleSave"
          v-show="activeStep === 4">完成</el-button>
      </div>
      <el-button
        style="margin-top: 12px;"
        @click="activeStep < 4 ? activeStep++ : activeStep = 4"
        v-show="activeStep < 4"
      >下一步</el-button>
    </el-form>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import CourseImage from '@/views/course/components/CourseImage.vue'
import TextEditor from '@/components/TextEditor.vue'
import { saveOrUpdateCourse, getCourseById } from '@/services/course'
import moment from 'moment'

export default Vue.extend({
  name: 'CreateOrUpdateCourse',
  components: {
    CourseImage,
    TextEditor
  },
  props: {
    isEdit: {
      type: Boolean,
      default: false
    },
    courseId: {
      type: [String, Number]
    }
  },
  data () {
    return {
      activeStep: 0,
      steps: [
        {
          title: '基本信息',
          icon: 'el-icon-edit'
        },
        {
          title: '课程封面',
          icon: 'el-icon-edit'
        },
        {
          title: '销售信息',
          icon: 'el-icon-edit'
        },
        {
          title: '秒杀活动',
          icon: 'el-icon-edit'
        },
        {
          title: '课程详情',
          icon: 'el-icon-edit'
        }
      ],
      activityCourse: false,
      course: {
        // id: 0,
        courseName: '',
        brief: '',
        teacherDTO: {
          // id: 0,
          // courseId: 0,
          teacherName: '',
          teacherHeadPicUrl: '',
          position: '',
          description: ''
        },
        courseDescriptionMarkDown: '',
        price: 0,
        discounts: 0,
        priceTag: '',
        discountsTag: '',
        isNew: true,
        isNewDes: '',
        courseListImg: '',
        courseImgUrl: '',
        sortNum: 0,
        previewFirstField: '',
        previewSecondField: '',
        status: 0, // 0：未发布，1：已发布
        sales: 0,
        activityCourse: false, // 是否开启活动秒杀
        activityCourseDTO: {
          // id: 0,
          // courseId: 0,
          beginTime: '',
          endTime: '',
          amount: 0,
          stock: 0
        },
        autoOnlineTime: ''
      }
    }
  },
  created () {
    if (this.isEdit) {
      this.loadCourse()
    }
  },
  methods: {
    async loadCourse () {
      const { data } = await getCourseById(this.courseId)
      const { activityCourseDTO } = data.data
      if (activityCourseDTO) {
        activityCourseDTO.beginTime = moment(activityCourseDTO.beginTime).format('YYYY-MM-DD')
        activityCourseDTO.endTime = moment(activityCourseDTO.endTime).format('YYYY-MM-DD')
      }
      this.course = data.data
    },
    async handleSave () {
      const { data } = await saveOrUpdateCourse(this.course)
      if (data.code === '000000') {
        this.$message.success('保存成功')
        this.$router.push('/course')
      } else {
        this.$message.error('保存失败')
      }
    }
  }
})
</script>

<style lang="scss" scoped>
</style>
