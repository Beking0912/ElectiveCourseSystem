<template>
  <div id="chooseClass">
    <h1>学生选课</h1>
  <div class="search-nox">
    <el-input v-model="input" class="input-with-select" placeholder="请输入内容">
      <el-select v-model="select" slot="prepend" class="aa" placeholder="请选择条件">
        <el-option
                v-for="item in options"
                :key="item.value"
                :label="item.label"
                :value="item.value">
        </el-option>
      </el-select>
      <el-button slot="append" icon="el-icon-search" @click="searchCourse(select,input)"></el-button>
    </el-input>
  </div>

    <div class="main">
      <el-row class="course_box card" :gutter="20">
        <div
          class="title"
        >当前可选课程范围：{{$store.state.department}} | {{$store.state.studentSemester}}课程</div>

        <el-col :md="12" :lg="8" :xl="6" v-for="item in courseData" :key="item.courseID">
          <div class="course" @click="choose(item.courseID)" :class="{choose:(item.courseID == chooseCourse)}">
            <div class="group">
              <div class="name">{{item.courseName}}</div>
            </div>
            <!--<div class="group">-->
              <!--<div class="departmentName">{{item.department}}</div>-->
            <!--</div>-->
            <div class="group">
              <div class="credit">{{item.teacherName}}</div>
              <div class="credit">{{item.credit}}学分</div>
              <div class="departmentName">{{item.departmentName}}</div>
              <div class="departmentName">&nbsp;&nbsp;{{item.classroom}}</div>
              <div class="departmentName">&nbsp;&nbsp;总量:{{item.capacity}}</div>

            </div>
          </div>
        </el-col>
        <transition>
          <el-button
            type="success"
            icon="el-icon-check"
            circle
            v-if="flag && chooseCourse != ''"
            @click="submit"
          ></el-button>
        </transition>
      </el-row>
      <div class="class_box card" style="overflow: visible">
        <div class="header">
          <div class="day">周一</div>
          <div class="day">周二</div>
          <div class="day">周三</div>
          <div class="day">周四</div>
          <div class="day">周五</div>
        </div>
        <div class="choose_card">
          <div class="class" v-for="(item,index) in classTable" :key="index">
            <div class="class_inner" v-if="item.timeID == ''"></div>
            <div class="ban" :class="{choose:item.choose, error:item.error}" v-if="item.timeID != ''"></div>
          </div>
        </div>

        <transition>
          <div class="card tag" v-if="chooseCourse == ''" style="background:#409EFF">请选择左侧您想要选课的课程</div>
        </transition>

        <transition>
          <div class="card tag" v-if="flag == false" style="background:#F56C6C">该课程与已选课程时间冲突，无法选课</div>
        </transition>

        <transition>
          <div
            class="card tag"
            v-if="flag == true && chooseCourse != ''"
            style="background:#67C23A"
          >本节课为可选状态</div>
        </transition>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      input: '',
      options: [{
        value: '4',
        label: '按课程编号'
      }, {
        value: '5',
        label: '按课程名称'
      }, {
        value: '6',
        label: '按教师名称'
      }],
      select: '',
      classData: [],
      courseData: [],
      chooseCourseClassData: [],
      classTable: [],
      chooseCourse: "",
      flag: true
    };
  },
  methods: {
    searchCourse(select, keyword) {
      if (select =='4'){
        this.axios
                .get("/searchCourseByCourseID?courseID=" + keyword)
                .then(res => {
                  if (res.data.code != 2) {
                    this.courseData = res.data.data;
                    console.log(this.courseData)
                  }
                })
                .catch(err => {
                  console.log(err);
                  this.$message("查询失败，该课程不存在请重新输入！");
                });
      }
      else if(select =='5'){
        this.axios
                .get("/searchCourseByCourseName?courseName=" + keyword)
                .then(res => {
                  if (res.data.code != 2) {
                    this.courseData = res.data.data;
                  }
                })
                .catch(err => {
                  console.log(err);
                  this.$message("查询失败，该课程不存在请重新输入！");
                });
      }
      else if(select =='6'){
        this.axios
                .get("/searchCourseByTeacherName?teacherName=" + keyword)
                .then(res => {
                  if (res.data.code != 2) {
                    this.courseData = res.data.data;
                  }
                })
                .catch(err => {
                  console.log(err);
                  this.$message("查询失败，该课程不存在请重新输入！");
                });
      }
      else {
        this.$message("查询失败，该课程不存在请重新输入！");
      }

    },
    getStudentClass() {
      this.axios
        .get("/getStudentClass?studentID=" + this.$store.state.studentID)
        .then(res => {
          if (res.data.code == 1) {
            this.classData = res.data.data;
            this.parseData();
          }
        })
        .catch(err => {
          console.log(err);
          this.$message("服务器无法连接");
        });
    },
    getCourseData() {
      this.axios
        .get(
          `/getChooseCourseList?departmentName=${
            this.$store.state.department
          }&semester=${this.$store.state.studentSemester}`
        )
        .then(res => {
          if (res.data.code == 1) {
            this.courseData = res.data.data;
          }
        })
        .catch(err => {
          console.log(err);
          this.$message("服务器无法连接，获取学生可选课程失败");
        });
    },
    // parseData:将获取到的classData（学生已选课节）填充数据，转存到classTable中
    parseData() {
      let data = this.classData;
      let finalData = [];
      for (let i = 0; i < 55; i++) {
        finalData[i] = {
          timeID: "",
          courseID: "",
          day: "",
          time: "",
          classroom: "",
          courseName: ""
        };
      }
      for (let i in data) {
        let day = parseInt(data[i].day);
        let time = parseInt(data[i].time);
        let index = (day - 1) * 4 + time - 1;
        finalData[index] = data[i];
      }
      this.classTable = finalData;
    },
    // 连带执行chooseClassToClassData
    choose(courseID) {
      this.chooseCourse = courseID;
      console.log(this.courseID);
      this.axios
        .get("/getClassByCourse?courseID=" + courseID)
        .then(res => {
          if (res.data.code == 1) {
            this.chooseCourseClassData = res.data.data;
            this.chooseClassToClassData();
          }
        })
        .catch(err => {
          console.log(err);
          this.$message("服务器无法连接，获取选择课程课节失败");
        });
    },
    chooseClassToClassData() {
      // 1. 清空classTable数据
      this.classTable = [];
      // 2. 调用parseData填充数据
      this.parseData();
      this.flag = true;
      // =====初始化课程表=======

      for (let i in this.chooseCourseClassData) {
        // 3. 为chooseCourseClassData添加choose属性
        this.chooseCourseClassData[i].choose = true;
        let day = this.chooseCourseClassData[i].day;
        let time = this.chooseCourseClassData[i].time;
        // 5. 将chooseCourseClassData与classData中的数据做比对，如果比对冲突，将this.flag设置为false，并且将冲突的对象添加error: true字段
        this.classData.some(item => {
          if (item.day == day && item.time == time) {
            this.chooseCourseClassData[i].error = true;
            this.flag = false;
            return true;
          }
        });
        // 4. 将chooseCourseClassData中的数据填充到classTable
        day = parseInt(day);
        time = parseInt(time);
        let index = (day - 1) * 4 + time - 1;
        this.classTable[index] = this.chooseCourseClassData[i];
      }
    },
    submit() {
      let obj = {
        studentID: this.$store.state.studentID.toString(),
        courseID: this.chooseCourse.toString()
      };
      this.axios
        .post("/chooseCourse", obj)
        .then(res => {
          if (res.data.code == 1) {
            this.$message.success("选课成功");
            this.$router.push("/student");
          }
        })
        .catch(err => {
          console.log(err);
          this.$message("服务器无法连接");
        });
    }
  },
  mounted() {
    this.getStudentClass(); //连带执行parseData
    this.getCourseData();
  }
};
</script>

<style lang='scss' scoped>
  .el-select .el-input {
    width: 130px;
  }
  .search-nox{
    padding: 20px;
  }
  .input-with-select .el-input-group__prepend {
    background-color: #fff;
  }
.aa{
  width:120px;
}

  .choose {
  background-color: #67c23a !important;
  border-color: #67c23a !important;
  color: white !important;
}
.error {
  background-color: #f56c6c !important;
  border-color: #f56c6c !important;
  color: white !important;
}
.main {
  position: relative;
  display: flex;

  .course_box {
    flex: 1 1 auto;
    margin-right: 20px !important;
    margin-left: 0px !important;
    padding-left: 10px;
    padding-right: 10px;
    padding-bottom: 0px;
    .title {
      color: white;
      background-color: #409eff;
      margin: -20px -10px;
      margin-bottom: 20px;
      padding: 20px;
      border-radius: 10px 10px 0 0;
    }
    .course {
      margin-bottom: 20px;
      padding: 20px;
      border-radius: 10px;
      box-sizing: border-box;
      border: 3px solid rgba(0, 0, 0, 0.12);
      cursor: pointer;
      font-size: 14px;
      color: #606266;
      .group {
        display: flex;
        line-height: 1.7;
        .name {
          font-size: 20px;
          font-weight: bold;
        }
        .credit {
          margin-right: 10px;
        }
      }
    }
    .el-button {
      width: 50px;
      height: 50px;
      position: absolute;
      bottom: -25px;
      right: 30px;
    }
  }
  .class_box {
    width: 290px;
    height: 640px;
    .header {
      display: flex;
      .day {
        width: 20%;
        font-size: 14px;
        text-align: center;
      }
    }
    .choose_card {
      display: flex;
      flex-direction: column;
      flex-wrap: wrap;
      height: 640px;
      .class {
        width: 20%;
        height: 55px;
        padding: 10px;
        box-sizing: border-box;
        .class_inner {
          height: 100%;
          border-radius: 5px;
          border: 2px solid rgba(0, 0, 0, 0.2);
        }
        .ban {
          height: 100%;
          border-radius: 5px;
          border: 2px solid rgba(0, 0, 0, 0.1);
          background: rgba(0, 0, 0, 0.2);
        }
      }
    }
  }
  .tag {
    margin: 40px -20px;
    color: white;
  }
}
</style>
