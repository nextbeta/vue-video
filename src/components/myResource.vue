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
						<span>我的视频</span>
					</h4>
				</div>
				<div class="panel-body" style="width: 100%; padding-top: 5px;">
					<div id="artMovie" class="col-md-12 movie_container">
						<div v-for="(movie,index) in myMoviesTmp" :meta="movie" :key="index"  class="col-md-2 col-xs-12 moviesClass">
							<p class="deleteVideoClass" @click="myconfirmDelete(movie.id)">删除</p>
							<a v-on:click="getVideos(movie.id)">
								<img :src="ftpIP + movie.imgpath" class="imgClass">
							</a>
							<p class="movetitleClass">{{movie.title}}</p>
							<p class="glyphicon glyphicon-eye-open seeImage">
								<span style="margin-left: 10px;">{{movie.click}}</span>
							</p>
							<p class="discriptionClass" :title="movie.description">{{movie.description}}</p>
							<p class="glyphicon glyphicon-align-left seeImage">
								<span style="margin-left: 10px;">{{movie.commentsnumber}}</span>
							</p>
						</div>
					</div>
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
				myMoviesTmp: {},
				total: 1, // 记录总条数
				pageSize: 24, // 每页显示条数
				current: 1, // 当前的页数
				username: "",
			}
		},
		created() {
			this.username = this.getCookie("username");
			this.initData();
			this.pagechange(1);
		},
		methods: {
			initData() {
				this.sessionToken = this.getCookie("sessionToken");
			},
			pagechange: function (currentPage) {
				let server = this.server;
				var that = this;
                var sessionToken = that.sessionToken;
				
				$.ajax({
					type: 'post',
					url: server + "/videos/findMyOwnVideos",
					beforeSend: function(XMLHttpRequest) {
						XMLHttpRequest.setRequestHeader("Authorization",sessionToken);
					}, 
					data: {
						"pageSize": that.pageSize,
						"currentPage": currentPage,
					},
					success: function (result) {
						if (result.code == 200) {
							that.myMoviesTmp = result.data.items;
							that.total = result.data.totalPages;
						}else{
							result = JSON.parse(result);
							if(result.code == 402){
								that.expireLogin();
							}
							that.$layer.msg(result.message);
						}
					}
				})
			},
			myconfirmDelete: function(videoId) {
				if(confirm("确定删除") == true){
					var server = this.server;
					var layer = this.$layer;
					var that = this;
					var sessionToken = that.sessionToken;
					
					$.ajax({
						type: "post",
						url: server + '/videos/deleteVideo',
						data: {
							"videoId": videoId,
						},
						beforeSend: function(XMLHttpRequest) {
							XMLHttpRequest.setRequestHeader("Authorization",sessionToken);
						},
						contentType: "application/x-www-form-urlencoded",
						success: function (result) {
							that.$layer.msg(result.message);
						}
					
					});
				}
			},
			getVideos: function (videoId) {
				this.$router.push({
					name: 'videoPlay',
					params: {
						//username: this.username,
						videoId: videoId,
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

	.movie_container {
		min-height: 710px;
		padding: 0;
		margin-right: 5px;
	}

	.moviepanel {
		min-height: 736px;
		width: 100%;
	}

	.imgClass {
		cursor: pointer;
		BORDER-TOP-WIDTH: 0px;
		BORDER-LEFT-WIDTH: 0px;
		FILTER: progid:DXImageTransform.Microsoft.RevealTrans (duration=2, transition=23);
		BORDER-BOTTOM-WIDTH: 0px;
		BORDER-RIGHT-WIDTH: 0px;
		width: 100%;
		height: 73%;
	}
	
	.deleteVideoClass{
		cursor: pointer;
		height: 16px;
		width: 75%;
		font-size: 12px;
		font-weight: bold;
		float: left;
	}

	.movetitleClass {
		cursor: pointer;
		height: 16px;
		width: 75%;
		overflow: hidden;
		white-space: nowrap;
		text-overflow: ellipsis;
		line-height: 16px;
		text-align: left;
		margin-top: 5px;
		font-size: 12px;
		font-weight: bold;
		float: left;
	}

	.discriptionClass {
		height: 15px;
		width: 75%;
		overflow: hidden;
		white-space: nowrap;
		text-overflow: ellipsis;
		line-height: 15px;
		text-align: left;
		color: #5C5C5C;
		float: left;
		font-size: 11px;
	}

	.moviesClass {
		height: 180px;
		margin-top: 5px;
	}

	.seeImage {
		height: 15px;
		line-height: 15px;
		width: 25%;
		color: #5C5C5C;
		font-size: 10px;
	}
</style>
