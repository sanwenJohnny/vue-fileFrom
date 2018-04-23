# vue-fileFrom
vue+jq.form.js  用于文件上传下载的组件


<template>
	<div class="file-btn-wrap">
		<form ref="formData" :id="formName" :action="url" enctype="multipart/form-data" :method="method" :class="'file-btn-' + this.type">
			<label v-show="!this.uploading" class="file-btn-label">
				<input v-show="type == 'import'" :id="fileName" @change="fileChange" type="file" name="file" accept="application/vnd.ms-excel,application/vnd.openxmlformats-officedocument.spreadsheetml.sheet">
			</label>
			<input v-for="(value, key) in params" :key="key" :name="key" :value="value" type="text">
		</form>
		<el-button v-show="type == 'import'" :loading="uploading" :type="btntype" icon="el-icon-dh-import" size="small">{{text ? text : '批量导入'}}</el-button>
		<el-button v-show="type == 'export'" @click="handleExport" :type="btntype" icon="el-icon-dh-export" size="small">{{text ? text : '导出'}}</el-button>
	</div>
</template>

<script>

import consts from '@/config/consts.js'
import config from '@/config/config.js'

export default {
	name: 'my-filehandler',
	data () {
		return {
			uploading: false
		}
	},
	props: {
		//typr 导入还是导出
		type: {
			default: "export"
		},
		action: {
			default: ""
		},
		//form的名字
		name: {
			default: ""
		},
		//上传其余参数
		params: {
			default: function() {
				return {}
			}
		},
		//上传下载方法 get/post
		method:{
			default:'post'
		},
		//按钮名称
		text: {
			default: ""
		},
		//按钮样式
		btntype: {
			default: ""
		},
		//不需要提示直接导出
		confirmBeforeExport: {
			default: true
		}
	},
	watch: {
		params: function() {
			// console.log(this.params);
		}

	},
	computed: {
		formName() {
			return this.type == "import" ? "form_" + this.type + "_" + this.name : "";
		},
		fileName() {
			return this.type == "import" ? "file_" + this.type + "_" + this.name : "";
		},
		url() {
			return this.action ? config.host.rfid + this.action : '';
		}
	},
	methods : {
		beforeTransfer(file) {
			const isExcel = file.type === "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet" || file.type === "application/vnd.ms-excel";

			if (!isExcel) {
				this.$message.error("文件只能是xls/xlsx格式");
			}

			return isExcel;
		},
		fileChange(e) {
			var self = this;
			this.uploading = true;
			let file = e.target.files[0];
			// console.log(file);
			if (!this.beforeTransfer(file)) {
				this.uploading = false;
				return;
			}
			$("#" + this.formName).ajaxSubmit({
				success: function(response) {
					self.uploading = false;
					if (response.success) {
						self.$message({
							message: '文件上传成功！',
							type: "success",
							duration: "1000"
						});
						self.$emit("onResponse", response.data);
					}else{
						self.$message({
							message: response.errMsg,
							type: "error",
							duration: "1000"
						});
					}
				},
				error: function(err) {
					self.uploading = false;
					self.$message({
						message: err.errMsg,
						type: "error",
						duration: "1000"
					});
					console.log(err.errMsg);
				}
			});
			e.target.value = "";
		},
		handleExport() {
			let ids = this.params.tableCheckedIds;
			if (!this.confirmBeforeExport) {
				this.$refs.formData.submit();
				return;
			}
			var msg = "";
			if (!ids || ids.length == 0) {
				msg = consts.msg.exportAll;
			}
			else {
				msg = consts.msg.exportChecked;
			}
			this.$confirm(msg, "提示", {
					confirmButtonText: "确定",
					cancelButtonText: "取消",
					type: "info"
				}).then(() => {
					this.$refs.formData.submit();
				}).catch(() => {
					return;
				});
		}
	},
	mounted() {
		// console.log("eltree loaded");
	},
	components: {

	}
}
</script>
