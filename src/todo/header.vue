<template>
	<div class="">
		<el-row class="rowHeader">
			<el-col :span="6">
				<div class="grid-content left_top" style="border:0px solid black;">

					<el-dropdown class="titleItem" show-timeout=100 @command="selectProject">
						<span class="el-dropdown-link" style="color:#ffffff;border:0px solid black;">
							{{project.name}}<i class="el-icon-arrow-down el-icon--right"></i>
						</span>

						<el-dropdown-menu slot="dropdown">
							<el-dropdown-item
								v-for="project in projectList"
								:key="project.objectId"
								:command="project.objectId"
								>
								{{project.name}}
							</el-dropdown-item>
						</el-dropdown-menu>

					</el-dropdown>
					<!-- <el-button icon="el-icon-search" size="mini" circle style="margin-left:16px;"></el-button> -->
					<el-input v-model="input" placeholder="输入搜索" prefix-icon="el-icon-search" size="small" style="margin-left:16px;width:180px;display:none;"></el-input>
				</div>
				

			</el-col>
			<el-col :span="12">
				<div class="grid-content center_title" style="border:0px solid black;">
					<span style="font-family: Tahoma;font-size:18px;">柳叶</span>
				</div>
			</el-col>
			<el-col :span="6"  >
				<div class="grid-content right_top" >
						<el-tooltip effect="dark" content="关于" placement="bottom">
							<i
								class="headIcon el-icon-about"
								@click="showAboutApp"/>

						</el-tooltip>

						<i
							class="headIcon el-icon-add"
							@click="addProject"/>

						<el-dropdown class="titleItem" @command="handleCommand" show-timeout=100>
							<span class="el-dropdown-link" style="color:#ffffff;">
								{{this.user.nickname}}<i class="el-icon-arrow-down el-icon--right"></i>
							</span>
							<el-dropdown-menu v-show="this.user != null" slot="dropdown">
								<el-dropdown-item command="profile">个人信息</el-dropdown-item>
								<el-dropdown-item command="dashboard" v-show="isSuperUser">数据面板</el-dropdown-item>
								<el-dropdown-item command="logout">注销</el-dropdown-item>
							</el-dropdown-menu>
						</el-dropdown>
				</div>
			</el-col>
		</el-row>
		<!-- <loading-block
            v-if="showLoading"
            :done="doneLoading"
            :failed="loadFailed"/> -->
	</div>



</template>
<script>
	import LoadingBlock from './LoadingBlock.vue';

  export default {
		props:{
			project: {
				type: Object,
				required: true
			},
			user: {
				type: Object,
				required: true
			},
			settingLean: {
				type: Object,
				required: true
			},
			isSuperUser: {
				type: Boolean,
				required: true,
				default:false
			}
		},
		data(){
			return{
				projectList:[]
			}
		},
		mounted:function(){

		},
		computed:{
			showLoading() {
            return true;
        },
        doneLoading() {
            return false;
        },
        loadFailed() {
            return false;
        },
		},
		components: {
		   LoadingBlock
		},
    methods: {
      showAboutApp() {
				console.log("showAbout")
				this.$emit('aboutAction')
			},
      gotoOpinion() {
				window.open('http://support.qq.com/products/30228');
			},
      gotoApi() {
				window.open('https://gudong.name/2018/09/07/todo-api.html');
			},
      updateProjectList(projectList) {
				this.projectList = projectList
			},
      appendProject(project) {
				console.log("add to list "+project.name);
				this.projectList.push(project)
			},
      addProject() {
				if(!this.isSuperUser && this.projectList.length>=5){
					this.$message({
            type: 'info',
            message: '非高级用户只能创建不超过5个项目面板'
          });
					return ;
				}
				var last = this.projectList[this.projectList.length-1]
				var priority = 0
				if(last != undefined){
					priority = last.priority+1
				}

				console.log('priority '+priority);
				const project = {
					'priority':priority
				}

				this.$emit('addProject',project)
			},
      selectProject(id) {
				console.log('command is '+id);
				const that = this
				var project = this.projectList.find(function(item){
					return item.objectId === id
				})
				this.$emit('selectProject',project,function(result){
					if(result){
						that.project = result
					}
				})
			},
      handleCommand(command) {
				if(command === 'logout'){
					this.showLogoutDialog()
				}
				if(command === 'dashboard'){
					this.$emit('dashboard')
				}
				if(command === 'profile'){
					this.$emit('profile')
				}
      },
			showLogoutDialog(){
				this.$confirm('确定要注销当前用户吗?', '提示', {
          confirmButtonText: '注销',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
					this.$emit('logout')
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '取消'
          });
        });
			}
    }
  }
</script>
<style lang="stylus" scoped>
.el-col {
	border-radius: 0px;
}

.left_top {
	background: transparent;
	padding-left:12px;
	display:flex;
	flex-direction: row;
	align-items:center;
	justify-content:flex-start;
}

.center_title {
	background: transparent;
	text-align: center;
}

.right_top {
	text-align: right;
	border:0px solid blue;
	display:flex;
	align-items:center;
	justify-content:flex-end;
}
.grid-content {
	border-radius: 0px;
}

.titleItem{
	vertical-align: middle;
	display: inline-block;
}

.headIcon{
	margin-right:10px;
	font-size:18px;
	border: 0px solid #000;
}

</style>
