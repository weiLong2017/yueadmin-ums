<template>
	<section>
		<el-row :gutter="20">
			<el-col :span="7">
				<el-card>
					<div class="account-info">
						<el-popover
						  ref="avatarPop"
						  placement="top"
						  title=""
						  width="200"
						  trigger="hover"
						  :content="avatarUrl ? '更换头像' : '上传头像'"
						  popper-class="text-center">
						</el-popover>
						<div class="avatar" v-popover:avatarPop @click="uploadAvatar">
							<img v-if="accountForm.avatar" :src="accountForm.avatar">
							<img v-else src="../../assets/img/avatar.gif" alt="头像">
						</div>
						<h3>{{accountForm.name}}</h3>
						<ul class="list-group">
							<li class="list-group-item active">
								<span>账户信息</span>
								<a href="javascript:;" title="编辑" @click="handleEdit"><i class="el-icon-edit"></i></a>
							</li>
							<li class="list-group-item">
								<label>邮箱</label>
								<span>{{accountForm.email}}</span>
							</li>
							<li class="list-group-item">
								<label>手机</label>
								<span v-if="accountForm.mobile">{{accountForm.mobile}}</span>
								<span v-else>未设置</span>
							</li>
							<li class="list-group-item">
								<label>性别</label>
								<span>{{accountForm.gender ? '男' : '女'}}</span>
							</li>
							<li class="list-group-item">
								<label>QQ</label>
								<span>{{accountForm.qq}}</span>
							</li>
							<li class="list-group-item">
								<label>生日</label>
								<span>{{accountForm.birthday}}</span>
							</li>
							<li class="list-group-item">
								<label>籍贯</label>
								<span>{{accountForm.origin}}</span>
							</li>
							<li class="list-group-item">
								<label>部门</label>
								<span>{{accountForm.organize}}</span>
							</li>
						</ul>
					</div>
				</el-card>
			</el-col>
			<el-col :span="17">
				<el-card class="card-primary">
					<div slot="header">
						<span>公司动态</span>
					</div>
				</el-card>
			</el-col>
		</el-row>
		<!-- 头像上传 -->
		<el-dialog :visible.sync="avatarVisible" title="头像上传">
			<el-upload
			  class="uploader avatar-uploader"
			  action="https://jsonplaceholder.typicode.com/posts/"
			  accept="image/jpeg, image/png"
			  :show-file-list="false"
			  :on-success="handleAvatarSuccess"
			  :before-upload="beforeAvatarUpload">
			  <img v-if="avatarUrl" :src="avatarUrl">
			  <i v-else class="el-icon-plus uploader-icon"></i>
			</el-upload>
			<div slot="footer">
				<el-button @click="avatarVisible = false">取消</el-button>
				<el-button type="primary" @click="uploadSubmit">确定</el-button>
			</div>
		</el-dialog>
		<!-- 账户信息编辑 -->
		<el-dialog :visible.sync="accountFormVisible" title="账户信息编辑">
			<el-row>
				<el-col :span="16" :offset="4">
					<el-form :model="accountForm" ref="accountForm" :rules="rules" label-width="120px">
						<el-form-item label="姓名：" prop="name">
							<el-input v-model="accountForm.name"></el-input>
						</el-form-item>
						<el-form-item label="性别：" prop="gender">
							<el-radio class="radio" v-model="accountForm.gender" :label="1">男</el-radio>
  						<el-radio class="radio" v-model="accountForm.gender" :label="0">女</el-radio>
						</el-form-item>
						<el-form-item label="QQ：" prop="qq">
							<el-input v-model="accountForm.qq"></el-input>
						</el-form-item>
						<el-form-item label="生日：">
							<el-date-picker
					      v-model="accountForm.birthday"
					      type="date"
					      placeholder="选择日期"
					      :picker-options="pickerOptions"
					      style="width: 100%"
					      @change="dateChange">
					    </el-date-picker>
						</el-form-item>
						<el-form-item label="身份证号：" prop="idcard">
							<el-input v-model="accountForm.idcard"></el-input>
						</el-form-item>
						<el-form-item label="身份证正面：" prop="idcard">
							<el-upload
							  class="uploader idcard-uploader"
							  action="https://jsonplaceholder.typicode.com/posts/"
							  accept="image/jpeg, image/png"
							  :show-file-list="false"
							  :on-success="handleIdCardFrontSuccess"
							  :before-upload="beforeAvatarUpload">
							  <img v-if="idcardFrontUrl" :src="idcardFrontUrl">
							  <i v-else class="el-icon-plus uploader-icon"></i>
							</el-upload>
						</el-form-item>
						<el-form-item label="身份证背面：" prop="idcard">
							<el-upload
							  class="uploader idcard-uploader"
							  action="https://jsonplaceholder.typicode.com/posts/"
							  accept="image/jpeg, image/png"
							  :show-file-list="false"
							  :on-success="handleIdCardBackSuccess"
							  :before-upload="beforeAvatarUpload">
							  <img v-if="idcardBackUrl" :src="idcardBackUrl">
							  <i v-else class="el-icon-plus uploader-icon"></i>
							</el-upload>
						</el-form-item>
					</el-form>
				</el-col> 
			</el-row>
			<div slot="footer">
				<el-button @click="accountFormVisible = false">取消</el-button>
				<el-button type="primary" @click="submitForm">确定</el-button>
			</div>
		</el-dialog>
	</section>
</template>
<script>
	export default {
		data() {
			return {
				accountForm: {
					name: 'Transform',
					avatar: 'https://avatars1.githubusercontent.com/u/20084997?v=4&s=460',
					gender: 1,
					email: '2630243397@qq.com',
					birthday: '1992-08-08',
					qq: '12345678',
					origin: '山东-济南',
					location: '北京',
					mobile: '',
					idcard: '130110199208081234',
					partner: '悦视觉全球摄影',
					organize: '技术部',
					idcardPicFront: '',
					idcardPicBack: '',
				},
				avatarUrl: '',
				idcardFrontUrl: '',
				idcardBackUrl: '',
				loading: false,
				avatarVisible: false,
				accountFormVisible: false,
				rules: {
					name: [
						{ required: true, message: '请输入姓名', trigger: 'blur' },
					],
					birthday: [
						{ type: 'date', message: '请选择出生日期', trigger: 'change' },
					],
				},
				
			}
		},
		methods: {
			// 获取用户信息
			getAccountInfo() {
				this.accountForm = {
					name: 'Transform',
					avatar: '',
					gender: 1,
					email: '2630243397@qq.com',
					birthday: '1992-08-08',
					qq: '12345678',
					origin: '山东-济南',
					location: '北京',
					mobile: '',
					idcard: '130110199208081234',
					partner: '悦视觉全球摄影',
					organize: '技术部',
					idcardPicFront: '',
					idcardPicBack: '',
				}
				this.avatarUrl = this.accountForm.avatar;
			},
			// 图片上传校验
			beforeAvatarUpload(file) {
        const isJPG = file.type === 'image/jpeg' || file.type === 'png';
        const isLt2M = file.size / 1024 / 1024 < 2;
        if (!isJPG) {
          this.$message.error('上传头像图片只能是 JPG 或 PNG 格式!');
        }
        if (!isLt2M) {
          this.$message.error('上传头像图片大小不能超过 2MB!');
        }
        return isJPG && isLt2M;
      },
			// 头像上传成功
			handleAvatarSuccess(res, file) {
        this.avatarUrl = URL.createObjectURL(file.raw);
        this.accountForm.avatar = URL.createObjectURL(file.raw);
        this.$message.success('上传成功')
      },
      // 身份证正面上传成功
			handleIdCardFrontSuccess(res, file) {
        this.idcardFrontUrl = URL.createObjectURL(file.raw);
        this.accountForm.idcardPicFront = URL.createObjectURL(file.raw);
        this.$message.success('上传成功')
      },
      // 身份证背面上传成功
			handleIdCardBackSuccess(res, file) {
        this.idcardBackUrl = URL.createObjectURL(file.raw);
        this.accountForm.idcardPicBack = URL.createObjectURL(file.raw);
        this.$message.success('上传成功')
      },
      dateChange(val) {
      	console.log(val)
      	this.accountForm.birthday = val
      },
      pickerOptions: {
      	disabledDate(time) {
      		return time.getTime() < Date.now()
      	}
      },
      uploadAvatar() {
      	this.avatarVisible = true
      },
      // 头像提交
      uploadSubmit() {
      	this.avatarVisible = false
      	this.accountForm.avatar = this.avatarUrl
      	this.$message.success('更新成功')
      },
      // 账户信息编辑
      handleEdit() {
      	this.accountFormVisible = true
      },
      // 账户信息提交
      submitForm() {
      	this.$refs.accountForm.validate(valid => {
      		let data = Object.assign({}, this.accountForm)
      		console.log(data)

      	})
      }
		},
		mounted() {
			this.getAccountInfo()
		}
	}
</script>
<style scoped lang="scss">
	.account-info {
		h3 {
			margin: 10px 0;
			color: #337ab7;
			text-align: center;
		}
		.avatar {
			width: 200px;
			height: 200px;
			margin: auto;
			img {
				width: 100%;
				height: 100%;
				border: 1px solid #ccc;
				border-radius: 50%;
				cursor: pointer
			}
		}
	}
	.el-popover {
		text-align: center
	}
	.el-card {
		> div {
			color: #fff;
	    background-color: #337ab7;
	    border-color: #337ab7;
		}
	}
	.list-group-item {
		&.active {
			a {
				margin-left: 15px;
				color: #fff
			}
		}
	}
</style>