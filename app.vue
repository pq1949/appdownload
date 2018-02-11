<template>
  <div id="app">
    <br>
    <br>
    <el-row>
      <el-col :span="6">
        <br>
      </el-col>
      <el-col :span="18">

        <el-steps :space="230" :active="step">
          <el-step title="步骤 1" description="选择平台和模块名称"></el-step>
          <el-step title="步骤 2" description="填写本次上传的版本号"></el-step>
          <el-step title="步骤 3" description="选择需要上传的文件，支持拖拽上传和批量上传"></el-step>
          <el-step title="步骤 4" description="点击开始上传"></el-step>
        </el-steps>
        <br>
        <br>
        <div class="block">
          <span class="demonstration">选择平台和模块名称:</span>
          <el-cascader expand-trigger="hover" :options="options" v-model="selectedOptions" @change="handleChange" @active-item-change="getMenu"></el-cascader>
          <span class="demonstration">版本号:</span>
          <el-input icon="document" style="width:200px" v-model="version" :placeholder="versionNum" @change="versionChange"></el-input>
          <el-button icon="upload" style="margin-left: 10px;" type="primary" @click="submitUpload">开始上传</el-button>
        </div>
        <br>
        <el-input type="textarea" style="width:755px" :rows="5" placeholder="请输入版本描述信息(可选)" v-model="textareaContent">

        </el-input>
        <div style="margin: 5px;"></div>
        <br>
        <el-upload class="upload-demo" drag multiple ref="upload" action="/upload" :on-change="fileChange" :on-preview="handlePreview" :on-remove="handleRemove" :auto-upload="false" :data="param" :on-error="uploadFailed" :on-success="uploadSuccess">
          <i class="el-icon-upload"></i>
          <div class="el-upload__text">将文件拖到此处，或
            <em>点击上传</em>
          </div>
          <div slot="tip" class="el-upload__tip">只能上传crt/plist/ipa/apk文件，且不超过200M &nbsp; &nbsp;
            <a href="mailto:qianpei@sysnew.com?cc=jhyu@sysnew.com&subject=版本上传页面建议和反馈&body=你好!">建议和反馈</a>&nbsp; &nbsp;qianpei@sysnew.com </div>
        </el-upload>

      </el-col>

    </el-row>
    <br>

    <el-dialog title="上传信息" :visible.sync="dialogVisible" size="tiny" :before-close="handleClose">
      <p>平台：{{param.platform}}</p>
      <p>模块：{{param.id}}</p>
      <p>版本：{{version}}</p>
      <p>文件列表：
        <ul>
          <li v-for="item in fileList">{{item.name}}</li>
        </ul>
      </p>
      <p>描述信息：{{textareaContent}}</p>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitToServer">确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>
export default {
  data() {
    return {
      versionNum:'请输入版本号',
      textareaContent: '',
      versionObj: {
        upi_merqrc: {
          version: '',
          upt_ts: ''
        },
        nets: {
          version: '',
          upt_ts: ''
        },
        bbl_client: {
          version: '',
          upt_ts: ''
        },
        bbl_merchant: {
          version: '',
          upt_ts: ''
        }, mop: {
          version: '',
          upt_ts: ''
        }, upi_wallet: {
          version: '',
          upt_ts: ''
        }, bbl_demo: {
          version: '',
          upt_ts: ''
        }, tpn_demo: {
          version: '',
          upt_ts: ''
        }

      },
      currentDate: new Date(),
      dialogVisible: false,
      step: 1,
      version: '',
      fileList: [],
      param: {
        platform: '',
        id: '',
        version: ''
      },
      selectedOptions: [],
      options: [{
        value: 'android',
        label: 'android',
        children: []
      }, {
        value: 'ios',
        label: 'ios',
        children: []
      }]
    };
  },
  mounted() {
    this.getVersion();
  },
  methods: {

    submitToServer() {
      this.dialogVisible = false;
      this.param.version = this.version;
      this.param.versionDesc = this.textareaContent;
      this.$refs.upload.submit();

      //this.$refs.upload.clearFiles();
      //this.fileList = [];

    },

    handleClose(done) {
      this.dialogVisible = false;
    },
    submitUpload() {
      if (!this.param.platform || !this.param.id) {
        this.$notify({
          title: '警告',
          message: "请选择平台和模块名称!",
          type: 'warning'
        });
        return;
      }

      if (this.version == '') {
        this.$notify({
          title: '警告',
          message: "请填写本次上传的版本号!",
          type: 'warning'
        });
        return;
      }

      if (this.fileList.length == 0) {
        this.$notify({
          title: '警告',
          message: "请选择要上传的文件!",
          type: 'warning'
        });
        return;
      }
      //this.$refs.upload.submit();
      this.dialogVisible = true;
      console.log('submit')
    },
    handleRemove(file, fileList) {
      this.fileList = this.getReadyFile(this.$refs.upload.$data.uploadFiles);
      console.log('handleRemove')
      console.log(file);
      //console.log(fileList);
    },
    handlePreview(file, fileList) {
      // this.fileList = fileList;
      console.log('handlePreview')
      //console.log(file);
      // console.log(this.$refs.upload);
      // console.log(this.$refs.upload.$data.uploadFiles);
      // console.log(fileList);
      //this.getVersion();
    },
    handleChange() {
      //this.getVersion();
      console.log(this.fileList)
      console.log(this.selectedOptions);
      this.param.platform = this.selectedOptions[0];
      this.param.id = this.selectedOptions[1];
      this.step = 2;
      var num = this.versionObj[this.param.id].version || "unknown";
      this.versionNum="当前版本号:" + num;
      console.log(this.param);
    },
    versionChange() {
      console.log('versionChange');
      if (this.param.platform == '') {
        this.$notify({
          title: '警告',
          message: "请选择平台和模块名称!",
          type: 'warning'
        });
        return;
      }
      this.step = 3;
    },
    getReadyFile(list) {
      var tem = [];
      list.forEach(function(el) {
        if (el.status == 'ready') {
          tem.push(el);
        }
      });
      return tem;
    },
    fileChange(file, fileList) {
      console.log('fileChange');
      this.fileList = this.getReadyFile(fileList);
      console.log(file);
      console.log(fileList);
      if (this.param.platform == '') {
        this.$notify({
          title: '警告',
          message: "请选择平台和模块名称!",
          type: 'warning'
        });
        return;
      }

      if (this.version == '') {
        this.$notify({
          title: '警告',
          message: "请填写本次上传的版本号!",
          type: 'warning'
        });
        return;
      }
      if (this.fileList.length != 0) {
        this.step = 4;
      }
    },
    uploadSuccess(rsp, file, fileList) {
      console.log('uploadSuccess');
      console.log(rsp);
      if (rsp.success) {
        this.$notify({
          title: '成功',
          message: '恭喜你,上传成功!',
          type: 'success'
        });
        this.getVersion();
      } else {
        this.$notify({
          title: '失败',
          message: rsp.message || '上传失败',
          type: 'error',
          duration: 0
        });
      }
    },
    uploadFailed(err, file, fileList) {
      this.$notify({
        title: '失败',
        message: err.message || err || '上传失败',
        type: 'error',
        duration: 0
      });
    },
    getMenu(val) {
      //console.log('active item:', val);
      var me = this;
      this.$axios.get('/menu.json')
        .then(function(response) {
          //console.log(response);
          console.log(me.options)
          me.options[0].children = response.data;
          me.options[1].children = response.data;
        })
        .catch(function(error) {
          console.log(error);
        });
    },
    getVersion() {

      var me = this;
      this.$axios.get('/version.json')
        .then(function(response) {
          console.log(response.data);
          me.versionObj = response.data;

        })
        .catch(function(error) {
          console.log(error);
        });
    }
  }
}
</script>

<style scope>
.el-upload-dragger {
  background-color: #fff;
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  box-sizing: border-box;
  width: 755px;
  height: 180px;
  text-align: center;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}

.time {
  font-size: 12px;
  color: #999;
}

.bottom {
  margin-top: 13px;
  line-height: 12px;
}

.button {
  padding: 0;
  float: right;
}

.image {
  width: 160px;
  height: 160px;
  display: block;
}

.clearfix:before,
.clearfix:after {
  display: table;
  content: "";
}

.clearfix:after {
  clear: both
}
</style>
