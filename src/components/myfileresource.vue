<template>
	<div class="container" style="width: 100%;">
		<div class="row col-md-10 col-md-offset-1" style="height: 150px;">
			<!--头部的容器 里面加上组件 menus-->
			<menus v-bind:username="username"></menus>
		</div>
		<div class="row col-md-10 col-md-offset-1" style="min-height: 800px;margin-top: 30px;">
			<!-- content -->
			<div class="panel moviepanel">
				<div class="panel-heading">
					<h4 class="panel-title">
						<span>我的文件</span>
					</h4>
				</div>
				<div class="panel-body" style="width: 100%; padding-top: 5px;">
					<table class="table">
						<thead>
							<tr>
								<th class="col-md-6">名称</th>
								<th class="col-md-3">时间</th>
								<th class="col-md-1">下载次数</th>
								<th class="col-md-1">文件大小</th>
								<th class="col-md-1">下载</th>
							</tr>
						</thead>
						<tbody>
							<tr v-for="(files,index) in myFilesResource" :meta="files" :key="index">
								<td>{{files.filename}}</td>
								<td>{{files.createtime}}</td>
								<td>{{files.downloadnumber}}</td>
								<td>{{files.filesize == 0 ? 1:files.filesize}}M</td>
								<td></td>
							</tr>
						</tbody>
					</table>
				</div>
			</div>

			<pagination :total="total" :current-page='current' @pagechange="pagechange"></pagination>

		</div>
		<div class="row">
			<!-- 底部 -->
			<footers></footers>
		</div>
	</div>
</template>

<script>
	import menus from './header'; // * 导入menus组件
	import footers from './footer'; // * 导入 footer组件
	import pagination from './pagination'; // * 导入 pagination 分页

	export default {
		data() {
			return {
				ftpIP: this.GLOBAL.ftpIP,
				server: this.GLOBAL.server,
				sessionToken: this.GLOBAL.sessionToken,
				username: "",
				userId: "",
				myFilesResource: {},
				total: 1, // 记录总条数
				pageSize: 24, // 每页显示条数
				current: 1, // 当前的页数
			}
		},
		created() {
			this.initData();
			this.pagechange(1);
		},
		methods: {
			initData() {
				/* this.username = sessionStorage.getItem('username') ? sessionStorage.getItem('username') : localStorage.getItem(
					'username');
				this.userId = sessionStorage.getItem('userId') ? sessionStorage.getItem('userId') : localStorage.getItem(
					'userId');
				this.sessionToken =  sessionStorage.getItem('sessionToken') ? sessionStorage.getItem('sessionToken') : localStorage.getItem(
						'sessionToken'); */
				this.username = this.getCookie("username");
				this.sessionToken = this.getCookie("sessionToken");
			},
			pagechange: function (currentPage) {
				let server = this.server;
				var that = this;
				var sessionToken = that.sessionToken;

				$.ajax({
				/* 	headers: {
						"test":"1",
						"Authorization":sessionToken
					}, */
					type: 'post',
					url: server + "/videos/getFilesByUserId",
					beforeSend: function(XMLHttpRequest) {
						XMLHttpRequest.setRequestHeader("Authorization",sessionToken);
					}, 
					data: {
						"pageSize": that.pageSize,
						"currentPage": currentPage,
					},
					success: function (result) {
						try{
							if (result.code == 200) {
								that.myFilesResource = result.data.items;
								that.total = result.data.totalPages;
							}else{
								result = JSON.parse(result);
								if(result.code == 402){
									that.expireLogin();
								}
								that.$layer.msg(result.message);
							}
							
						}catch(e){
							console.log(e.message);
						}
						
					}
				})
			},
		},
		components: { // * 注册menus组件，让其可以在template调用
			menus,
			footers,
			pagination,
		}
	};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
	.panel-title {
		font-size: 18px;
		font-family: "arial black";
	}
</style>
