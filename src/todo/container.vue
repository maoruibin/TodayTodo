<template>
	<draggable class="dragList" :list="groupTodos" :options="{handle:'.topAction',ghostClass:'ghost',scroll: true,animation: 150,group:{ name:'groupList'}}"  @start="drag" @end="drop" >

		<Todo
			v-for="(group,index) in groupTodos"
			:key="group.objectId"
			:index="index"
			:isLastIndex="index == groupTodos.length -1"
			:group="group"
			:user="user"
			:allTags="allTags"
			ref="childTodo"
			@delete="deleteGroup"
			@showTodoDetail="showTodoDetail"
			@editGroup="showEditGroupDialog"
			@onFileGroup="fileGroup"
			@hideOtherInput="hideOtherInput"
			@appendGroup="appendGroup"
		/>

	</draggable>



</template>

<script >
import Todo from './todo.vue'
import draggable from 'vuedraggable'

let host = process.env.API_HOST
let api_version = process.env.API_VERSION

export default{
	props:{
		user:{
			type: Object,
			required: true
		}
	},
	data(){
		return{
			groupTodos:[],
			allTags:[],
      project:{},
			groupForAppend:{}
		}
	},
	components: {
		Todo,
		draggable
	},
	created() {
		
	},
	mounted:function(){
    if(!this.user){
      return
		}
		this.fetchAllTags();
  },
	
  methods:{
		//加载项目数据
		fetchProjectTodos(project,callback){

			this.project = project
			const loading = this.$loading({
	          lock: true,
	          text: '加载中,请稍等...',
	          spinner: 'el-icon-loading',
	          background: 'rgba(0, 0, 0, 0.7)'
	        });

	    const apiTodosAll = host+"/api/"+api_version+"/todos/projects/"+this.user.objectId+"/"+this.project.objectId
			console.log("select project is "+project.name);
			console.log("host "+apiTodosAll);

	    this.$http.get(apiTodosAll).then(response => {
	        this.groupTodos = response.body.data
					this.groupForAppend.priority = this.groupTodos.length
					this.groupTodos.push(this.groupForAppend)
	        console.log("len is "+this.groupTodos.length);
					loading.close();
					callback(1)
	      }, response => {
					loading.close();
					this.$message.error('加载数据出了点问题，请重试。('+response.status+"-"+response.statusText+")");
					callback(0)
				});

		},
		showTodoDetail(todo,filterTodos){
			this.$emit('showTodoDetail',todo,filterTodos)
		},
		drag:function(){
			console.log('drag');
		},
		drop:function(e){
			console.log('drop');
			const that = this;
			var len = this.groupTodos.length
			//优先级最低 0 最高为 size-1
			var updateList = []
			this.groupTodos.forEach(function(group, index, array){
					group.priority = String(index)
					// 不是最后一个
					if(index<len-1){
						var updateItem = {objectId:group.objectId,priority:group.priority}
						updateList.push(updateItem)
					}
			})

			const api = host+"/api/"+api_version+"/groups/batchEdit"
			const config = {
					headers : {
							'Content-Type' : 'application/x-www-form-urlencoded; charset=UTF-8'
					}
			}
			var formData = new FormData();
			formData.append('list', JSON.stringify(updateList));

			this.$http.post(api, formData, config).then(response => {
					//编辑完的 todo 结果
					const msg = response.body.msg;
					// 将编辑完的结果回调回去
					console.log(msg);
				}, response => {});


		},
		fetchAllTags(){
				this.user = JSON.parse(localStorage.getItem("user"))
				const apiTagsAll = host+"/api/"+api_version+"/tags/"+this.user.objectId
				this.$http.get(apiTagsAll).then(response => {
						this.allTags = response.body.data
						this.allTags.forEach(function(item){
								item.value = item.name
						})

						}, response => {}
				);
    }, 
    appendGroup:function(event){
			this.showAddGroupDialog();
    },

		showAddGroupDialog(){
			this.$prompt('', '请输入分组名', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          inputPattern: /[\u4e00-\u9fa5_a-zA-Z0-9]{1,6}/,
          inputErrorMessage: '格式不正确'
        }).then(({ value }) => {
					const group = {
						'name':value,
						'projectId':this.project.objectId,
						'priority':this.groupForAppend.priority,
						'userId':this.user.objectId
					}

					this.addNewGroup(group)
        }).catch(() => {

        });
		},

		showEditGroupDialog(group){
			this.$prompt('', '修改分组名称', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          inputValue: group.name,
          inputPattern: /[\u4e00-\u9fa5_a-zA-Z0-9]{1,6}/,
          inputErrorMessage: '格式不正确'
        }).then(({ value }) => {
					group.name = value
					this.editGroup(group)
        }).catch(() => {

        });
		},

		addNewGroup(group){
			const api = host+"/api/"+api_version+"/groups"
			var formData = new FormData();
			formData.append('name', group.name);
			formData.append('projectId', group.projectId);
			formData.append('priority', group.priority);
			formData.append('userId', group.userId);

			// POST /someUrl
		  this.$http.post(api, formData).then(response => {
				this.groupTodos.splice(this.groupTodos.length-1,0,response.body.data)
				this.$message({
					type: 'success',
					message: '插入新分组成功'
				});
		  }, response => {

		  });
		},
		// 更新 group
		updateGroup(group,callback){
			const api = host+"/api/"+api_version+"/groups"
			var formData = new FormData();

			for(var key in group) {
				if(!(group[key] instanceof Object)){
					console.log("update "+key+" to "+group[key]);
					formData.append(key, group[key]);
				}
			}

		  this.$http.put(api, formData).then(response => {
				this.groupTodos.splice(this.groupTodos.findIndex(item => item.objectId === group.objectId),1,response.body.data)
				callback(response.body.data,1)
		  }, response => {
				callback(null,-1)
		  });
		},
		fileGroup(group){
			const that = this
			this.updateGroup(group,function(result,flag){
				if(flag>0){
					that.groupTodos.splice(that.groupTodos.findIndex(item => item.objectId === group.objectId),1)
					that.$message({
						type: 'success',
						message: '已归档'+group.name
					});
				}else{

				}

			})
		},
		editGroup(group){
			const that = this
			this.updateGroup(group,function(){
				that.$message({
					type: 'success',
					message: '更新分组成功'
				});
			})
		},
		// 提供给父组件 让父组件隐藏改分组的输入表单
		hideOtherInput(group){
			// 隐藏其他正在显示的输入框
			this.$refs.childTodo.forEach(function(child){
					child.checkAndHideGroupInputForm(group.objectId);
			})
		},
		deleteGroup(objectId){
			const api = host+"/api/"+api_version+"/groups/"+objectId
			const config =
				{
				    headers : {
				        'Content-Type' : 'application/x-www-form-urlencoded; charset=UTF-8'
				    }
				}

			this.$http.delete(api,config).then(response => {
					console.log("result is "+response.body.data);
					this.groupTodos.splice(this.groupTodos.findIndex(group => group.objectId === objectId),1)

					this.$message({
						type: 'success',
						message: '删除成功!'
					});

				}, response => {});
		}
  }

}
</script>


<!-- 设置scoped 表示当前组件下的id只在当前组件起作用，不会跟其他组件引起冲突 -->
<style lang="stylus" scoped>

.dragList{
	 min-height: 10px;
}

</style>
