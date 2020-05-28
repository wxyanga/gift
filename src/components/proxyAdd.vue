<template>
	<div>
		<el-form :inline="true" label-width="100px">
			<el-form-item label="代理名：">
				<el-input v-model="form.proxyId" placeholder="输入代理名"></el-input>
			</el-form-item>
			<!-- <el-form-item label="VIP类型：">
				<el-select v-model="form.type">
					<el-option v-for="(item, index) in lvMap" :key="index" :label="item.label" :value="item.value"></el-option>
				</el-select>
			</el-form-item>-->
			<!-- <el-form-item label="注册时间：">
				<el-date-picker
					format="yyyy 年 MM 月 dd 日"
					value-format="yyyy-MM-dd"
					v-model="form.times"
					type="daterange"
					range-separator="至"
					start-placeholder="开始日期"
					end-placeholder="结束日期"
				>
				</el-date-picker>
			</el-form-item>-->
			<!-- <el-form-item label="充值范围：">
				<el-input style="width:100px" v-model="form.min" :controls="false" placeholder="最小金额"></el-input>
				<span style="margin:0 5px">至</span>
				<el-input style="width:100px" v-model="form.max" :controls="false" placeholder="最大金额"></el-input>
			</el-form-item>-->
			<el-form-item>
				<el-button type="primary" @click="getTableData">查询</el-button>
				<el-button type="primary" @click="addVipType">新增代理</el-button>
			</el-form-item>
		</el-form>
		<hr />
		<el-table :data="tableData">
			<!-- <el-table-column type="index" label="#"></el-table-column> -->
			<el-table-column label="代理电话" prop="user_id"></el-table-column>
			<!-- <el-table-column label="账户余额" prop="bal"> </el-table-column> -->
			<el-table-column label="代理商底价" prop="price"></el-table-column>
			<el-table-column label="推荐会员总余额" prop="bal_summ"></el-table-column>
			<el-table-column label="QQ号码" prop="qq"></el-table-column>
			<el-table-column label="E-mail" prop="email"></el-table-column>
			<el-table-column label="创建时间" prop="create_date"></el-table-column>
			<el-table-column label="操作">
				<template slot-scope="prop">
					<el-button @click="changeVIPType(prop.row)">编辑</el-button>
					<!-- <el-button @click="lookHistory(prop.row['user_id'])">查看修改历史</el-button> -->
				</template>
			</el-table-column>
		</el-table>
		<!-- <el-pagination
			@size-change="handleSizeChange"
			@current-change="handleCurrentChange"
			background
			:total="total"
			:page-size="pageSize"
			:page-sizes="pageSizes"
			:current-page="pageNum"
			layout="total,  prev, pager, next"
		>
		</el-pagination>-->
		<el-dialog title="密码重置成功" :visible="modifyStytus" @close="closeDialogModify">
			<div>
				<el-form :model="handleForm" ref="handleForm" label-width="150px">
					<el-form-item label="代理商新密码：">{{ modifyNewPass }}</el-form-item>
				</el-form>
			</div>
		</el-dialog>
		<common-dialogs
			:right-click="dialogConfirm"
			:left-click="closedDialog"
			:closed-dialog="closedDialog"
			class="cust-Dialog-01"
			:isCenter="$store.state.dialogOptions.slotName == 'modify'"
			:center-click="continueAdd"
			:center-button-type="'danger'"
			:centerButtonText="'重置密码'"
		>
			<template slot="add">
				<el-form :model="handleForm" :rules="handleFormRulesReg" ref="handleForm" label-width="150px">
					<!-- { userid: '', passWord: '', surePassword: '', qq: '', email: '', vstr: '' } -->
					<el-form-item label="代理电话：" prop="userid">
						<el-input v-model="handleForm.userid"></el-input>
					</el-form-item>
					<el-form-item label="登录密码：" prop="passWord">
						<el-input v-model="handleForm.passWord"></el-input>
					</el-form-item>
					<el-form-item label="确认登录密码：" prop="surePassword">
						<el-input v-model="handleForm.surePassword"></el-input>
					</el-form-item>
					<el-form-item label="代理QQ：" prop="qq">
						<el-input v-model="handleForm.qq"></el-input>
					</el-form-item>
					<el-form-item label="代理email：" prop="email">
						<el-input v-model="handleForm.email"></el-input>
					</el-form-item>
					<el-form-item label="代理商底价：" prop="price">
						<el-input-number
							controls-position="right"
							class="xzl_input_number"
							:controls="false"
							:min="0"
							v-model="handleForm.price"
						></el-input-number>
					</el-form-item>
				</el-form>
			</template>
			<template slot="history"></template>
			<template slot="modify">
				<el-form
					:model="handleForm"
					:rules="handleFormRulesRegMSG"
					ref="handleFormRulesRegMSG"
					label-width="150px"
				>
					<!-- { userid: '', passWord: '', surePassword: '', qq: '', email: '', vstr: '' } -->
					<el-form-item label="代理电话：">
						{{ handleForm.userid }}
						<!-- <el-input v-model="handleForm.userid"></el-input> -->
					</el-form-item>

					<el-form-item label="代理QQ：" prop="qq">
						<el-input v-model="handleForm.qq"></el-input>
					</el-form-item>
					<el-form-item label="代理email：" prop="email">
						<el-input v-model="handleForm.email"></el-input>
					</el-form-item>
					<el-form-item label="代理商底价：" prop="price">
						<el-input-number
							controls-position="right"
							class="xzl_input_number"
							:controls="false"
							:min="0"
							v-model="handleForm.price"
						></el-input-number>
					</el-form-item>
				</el-form>
			</template>
		</common-dialogs>
	</div>
</template>

<script>
	import mixin from 'static/mixins.js'
	import CommonDialogs from './CommonDialogs.vue'

	export default {
		components: {
			CommonDialogs
		},
		mixins: [mixin.pagination],
		name: 'login',
		data() {
			let sameRules = {
				userid: [
					{ required: true, message: '请输入用户名', trigger: 'blur' },
					{ min: 6, message: '请输入注册时使用的手机号码', trigger: 'blur' }
				],
				passWord: [
					{ required: true, message: '请输入密码', trigger: 'blur' },
					{ min: 6, message: '密码由至少6位数字或字母组成', trigger: 'blur' }
				],
				vstr: [{ required: true, message: '请输入验证码', trigger: 'blur' }]
			}
			let checkPassword = (rule, value, callback) => {
				if (!value) {
					return callback(new Error('确认密码不能为空'))
				}
				if (value !== this.handleForm.passWord) {
					return callback(new Error('确认密码与密码不同'))
				}
				if (value.length < 6) {
					return callback(new Error('确认密码由至少6位数字或字母组成'))
				}
				callback()
			}
			let checkUserName = (rule, value, callback) => {
				if (!value) {
					return callback(new Error('请使用手机号码注册'))
				}
				if (!/^1[3456789]\d{9}$/.test(value)) {
					return callback(new Error('手机号码填写有误，请重新输入'))
				}
				callback()
			}
			return {
				index: 0,
				systomConfig: systomConfig,
				timeStmp: '',
				titleName: ['登录', '快速注册'],
				formLogin: { userid: '', passWord: '', vstr: '', vid: '' },
				rulesLogin: {
					userid: [
						{ required: true, message: '请输入用户名', trigger: 'blur' },
						{ min: 6, message: '请输入注册时使用的手机号码', trigger: 'blur' }
					],
					passWord: [
						{ required: true, message: '请输入密码', trigger: 'blur' },
						{ min: 6, message: '密码由至少6位数字或字母组成', trigger: 'blur' }
					],
					vstr: [{ required: true, message: '请输入验证码', trigger: 'blur' }]
				},
				formReg: { userid: '', passWord: '', surePassword: '', qq: '', email: '', vstr: '', vid: '', price: '' },
				rulesReg: {
					userid: [
						{ validator: checkUserName, required: true, trigger: 'blur' }
						// { required: true, message: '请使用手机号码注册', trigger: 'blur' },
						// { min: 6, message: '用户名由至少6位数字或字母组成', trigger: 'blur' }
					],
					passWord: [
						{ required: true, message: '请输入密码', trigger: 'blur' },
						{ min: 6, message: '密码由至少6位数字或字母组成', trigger: 'blur' }
					],
					vstr: [{ required: true, message: '请输入验证码', trigger: 'blur' }],
					surePassword: [{ validator: checkPassword, required: true, trigger: 'blur' }],
					qq: [{ required: true, message: '请输入联系QQ', trigger: 'blur' }],
					email: [
						{
							required: true,
							type: 'email', //要检验的类型（number，email，date等）
							message: '请输入正确的邮箱地址',
							trigger: ['blur', 'change']
						}
					]
				},
				handleFormRulesReg: {
					userid: [
						{ validator: checkUserName, required: true, trigger: 'blur' }
						// { required: true, message: '请使用手机号码注册', trigger: 'blur' },
						// { min: 6, message: '用户名由至少6位数字或字母组成', trigger: 'blur' }
					],
					passWord: [
						{ required: true, message: '请输入密码', trigger: 'blur' },
						{ min: 6, message: '密码由至少6位数字或字母组成', trigger: 'blur' }
					],
					surePassword: [{ validator: checkPassword, required: true, trigger: 'blur' }],
					qq: [{ required: true, message: '请输入联系QQ', trigger: 'blur' }],
					email: [
						{
							required: true,
							type: 'email', //要检验的类型（number，email，date等）
							message: '请输入正确的邮箱地址',
							trigger: ['blur', 'change']
						}
					],
					price: [{ required: true, message: '请输入代理商每单分润', trigger: 'blur' }]
				},
				handleFormRulesRegMSG: {
					qq: [{ required: true, message: '请输入联系QQ', trigger: 'blur' }],
					email: [
						{
							required: true,
							type: 'email', //要检验的类型（number，email，date等）
							message: '请输入正确的邮箱地址',
							trigger: ['blur', 'change']
						}
					],
					price: [{ required: true, message: '请输入代理商每单分润', trigger: 'blur' }]
				},
				form: {
					proxyId: '',
					userId: '',
					type: '',
					times: ['', ''],
					min: '',
					max: ''
				},
				uploadForm: {
					bal: 0,
					price: 0,
					qq: '',
					email: ''
					// userType: ''
				},
				historyData: [],
				historyStytus: false,
				ModifyMoney: false,
				ModifyVip: false,
				ModifyType: false,
				modifyStytus: false,
				modifyQQ: false,
				modifyEmail: false,
				modifyNewPass: '',
				handleForm: {},
				lvMap: []
			}
		},
		created() {
			if (this.$route.params.step) this.index = this.$route.params.step
			this.getTimeStamp(false)
		},
		mounted() {
			this.getTableData()
		},
		methods: {
			getTableData() {
				let params = {
					proxyId: this.form.proxyId
				}
				this.$post(this.$API.PROXY_SEARCH, params, '').then(res => {
					this.tableData = res.map(it => {
						it.price = it.price > 0 ? it.price : 0
						return it
					})
				})
			},
			dialogConfirm() {
				let slotName = this.$store.state.dialogOptions.slotName
				if (slotName == 'add') {
					this.$refs.handleForm.validate(valid => {
						if (valid) {
							let params = this.handleForm
							this.$post(this.$API.PROXY_ADD, params, '添加成功').then(res => {
								this.handleForm = { userid: '', passWord: '', surePassword: '', qq: '', email: '', price: '' }
								this.getTableData()
							})
						}
					})
				} else if (slotName == 'modify') {
					// URL_KF_UPLOAD_USER
					this.$refs.handleFormRulesRegMSG.validate(valid => {
						if (valid) {
							let params = {
								userId: this.handleForm['user_id']
								// ...this.uploadForm
							}
							// if (this.uploadForm.bal >= 0) params.bal = this.uploadForm.bal
							// if (this.uploadForm.userType) params.userType = this.uploadForm.userType
							if (this.handleForm.price > 0) params.price = this.handleForm.price
							if (this.handleForm.qq != '') params.qq = this.handleForm.qq
							if (this.handleForm.email != '') params.email = this.handleForm.email
							this.$post(this.$API.URL_KF_UPLOAD_USER, params, '代理商信息修改成功').then(res => {
								this.handleForm = { userid: '', passWord: '', surePassword: '', qq: '', email: '', price: '' }

								this.getTableData()
							})
						}
					})
				}
			},
			closedDialog() {},
			addVipType() {
				this.handleForm = { userid: '', passWord: '', surePassword: '', qq: '', email: '', price: '' }
				this.$showDialog({ title: '添加代理', slotName: 'add' })
			},
			changeVIPType(row) {
				this.handleForm = { ...row, userid: row['user_id'] }
				this.$showDialog({ title: '代理信息修改', slotName: 'modify' })
			},
			goIndex() {
				this.$router.push('/')
			},
			getTimeStamp(isShowLoading = true) {
				this.$post(this.$API.URL_GET_CHECK, {}, '', isShowLoading).then(res => {
					if (this.index == 0) this.formLogin.vid = res.id
					else this.formReg.vid = res.id
					this.timeStmp = res.img
				})
			},
			changeForm(num) {
				this.index = num
				this.getTimeStamp()
				this.$nextTick(() => {
					this.$refs.formLogin && this.$refs.formLogin.clearValidate()
					this.$refs.formReg && this.$refs.formReg.clearValidate()
				})
			},
			userLogin() {
				this.$refs.formLogin.validate(valid => {
					if (valid) {
						this.$post(this.$API.URL_LOGIN, this.formLogin, '').then(
							res => {
								sessionStorage.setItem('loginState', true)
								sessionStorage.setItem('INFO', JSON.stringify(res))
								this.$store.commit('isLogin', sessionStorage.getItem('loginState'))
								this.$router.push('/home/')
							},
							() => {
								return this.getTimeStamp()
							}
						)
					} else {
						return false
					}
				})
			},
			userRegister() {
				this.$refs.formReg.validate(valid => {
					if (valid) {
						this.$post(this.$API.URL_REGISTER, this.formReg, '').then(
							() => {
								this.$message.success('注册成功，请登录！')
								this.formReg = { userid: '', passWord: '', surePassword: '', qq: '', email: '', vstr: '', price: '' }
								this.changeForm(0)
							},
							() => {
								return this.getTimeStamp()
							}
						)
					} else {
						return false
					}
				})
			},
			continueAdd() {
				this.$post(this.$API.URL_SYSTOEM_RESET_PASSWORD, { userId: this.handleForm['user_id'] }, '').then(res => {
					this.modifyNewPass = res['new_password']
					this.modifyStytus = true
				})
			},
			closeDialogModify() {
				this.modifyNewPass = ''
				this.modifyStytus = false
			}
		}
	}
</script>
<style>
	.xzl_input_number.el-input-number .el-input__inner {
		text-align: left;
	}
</style>
