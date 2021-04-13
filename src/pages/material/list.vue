<template>
	<div class="hello">
		<div v-if="showList">
			<!--		搜索框-->
			<div style="position: fixed;left: 0;right: 0;top: -1px;z-index: 1000; padding-top: 10px;">
				<van-field
						clearable
						size="small"
						style="width: calc(100% - 80px);display: inline-block;height: 33px;padding: 5px 10px;background-color: #EBEDF0; border-radius: 8px;"
						v-model="inputValue"
						placeholder="请输入素材名称"
				/>
				<span type="info" size="small"
				      style="width: 43px; display: inline-block;vertical-align: top;height: 34px;line-height: 34px;font-size: 14px;"
				      @click="queryName">搜索</span>
				<!--		选择分组-->
				<van-field
						readonly
						clickable
						name="picker"
						label="选择分组："
						v-model="groupName"
						@click="showPicker = true"
						placeholder="所有"/>
			</div>
			<!-- 图片预览 -->
			<view
					v-if="show"
					@click="cancelShow"
					class="uni-margin-wrap preview-box">
				<view
						class="swiper-item">
					<img
							:src="previewUrl"
							width="75%"
							class="img-info"/>
				</view>
			</view>
			<view class="web-back" v-if="showVideoPreview">
				<uni-nav-bar
						@clickLeft="cancelVideo">
					<view slot="default"
					      style="text-align: center;width: 100%;overflow: hidden;text-overflow: ellipsis;white-space: nowrap;">
						{{previewTitle}}
					</view>
					<view slot="left" style="margin-left:9px; cursor: pointer; color:#1989FA;">关闭</view>
				</uni-nav-bar>
			</view>
			<view class="web-view" v-if="showVideoPreview">
				<!-- 视频预览页面 -->
				<web-view :src="videourl" @click="cancelVideo"></web-view>
			</view>
			<!-- 分组 -->
			<van-popup v-model="showPicker" position="bottom">
				<div>
					<div style="height: 40px;line-height: 40px;color: #1989FA;padding: 0 10px;">
						<div style="float: left;cursor: pointer;" @click="handlerCancle">取消</div>
						<div class="chooseGroupName">{{this.$store.state.groupName}}
							<van-icon
									name="close"
									@click="clearGroup"
									v-show="this.$store.state.groupName != ''"
									style="margin-left: 5px;"/>
						</div>
						<div style="float: right;cursor: pointer;" @click="handlerOk">确定</div>
					</div>
					<div style="height: 200px;overflow-y: auto;">
						<Group :data="groupArr"></Group>
					</div>
				</div>
			</van-popup>
			<!--   style="position: absolute; top: 100px; width: 100%;"    -->
			<div style="position: fixed; top: 100px; width: 100%;height: calc(100% - 100px)">
				<div class="metarial-list" style="height: 100%; overflow-y: auto;">
					<van-pull-refresh v-if="fileType == 1"
					                  v-model="ishFiveLoading"
					                  success-text="刷新成功"
					                  @refresh="onRefresh"
					>
						<van-grid :gutter="10" :column-num="3" style="padding: 10px 10px;">
							<van-grid-item
									v-for="(item,index) in picList"
									:key="index">
								<img
										v-lazy="item.s_local_path"
										@click="previewPic(item.s_local_path, item.id)"
										class="img"
										v-if="item.s_local_path != null && item.s_local_path != ''"
										:style="height"/>
								<img
										v-lazy="item.local_path"
										@click="previewPic(item.local_path, item.id)"
										class="img"
										v-if="item.s_local_path == null || item.s_local_path == ''"
										:style="height"/>
								<span class="pic-text" @click="previewPic(index,item)">{{item.file_name}}</span>
							</van-grid-item>
						</van-grid>
					</van-pull-refresh>
					<view v-if="noMorePic && fileType == 1"
					      style="color: #969799;font-size: 14px;line-height: 50px;margin-top: -20px;">
						没有更多了
					</view>
					<van-loading size="24px" v-if="picLoading && fileType == 1" style="margin: -5px 20px 15px">加载中...
					</van-loading>
					<van-pull-refresh v-if="fileType == 4"
					                  v-model="ishFiveLoading"
					                  success-text="刷新成功"
					                  @refresh="onRefresh"
					>
						<van-list
								v-model="hFiveLoading"
								:finished="noMorehFive"
								finished-text="没有更多了"
								@load="loadingItem">
							<template v-for="(item,index) in hFiveList">
								<van-cell
										:key="index">
									<template slot="title">
										<img
												v-lazy="item.s_local_path"
												class="img2"
												v-if="item.s_local_path != null && item.s_local_path != ''"
												@click="h5Preview(item.jump_url,item)"/>
										<img
												v-lazy="item.local_path"
												class="img2"
												v-if="item.s_local_path == null || item.s_local_path == ''"
												@click="h5Preview(item.jump_url,item)"/>
										<div style="width: calc(100% - 65px);float: right;"
										     @click="h5Preview(item.jump_url,item)">
											<span class="list-text" v-html="item.file_name"
											      style="line-height: 18px;"></span>
											<span class="list-text2" v-html="item.content"
											      v-show="item.content != '' && item.content != null"></span>
											<span class="list-text2" v-if="!item.content" v-html="item.file_name"
											      style="line-height: 18px;"></span>
										</div>
									</template>
								</van-cell>
							</template>
						</van-list>
					</van-pull-refresh>
					<van-pull-refresh v-if="fileType == 5"
					                  v-model="isFileLoading"
					                  success-text="刷新成功"
					                  @refresh="onRefresh"
					>
						<van-list
								v-model="fileLoading"
								:finished="fileFinished"
								finished-text="没有更多了"
								@load="loadingItem"
						>
							<van-cell @click="openFile(item.local_path)"
							          v-for="(item,index) in fileList"
							          :key="index">
								<template slot="title">
									<img src="@/static/assets/doc.png" alt="" class="file-img"
									     v-if="item.extension == 'doc'">
									<img
											src="@/static/assets/docx.png"
											alt=""
											class="file-img"
											v-if="item.extension == 'docx'">
									<img src="@/static/assets/xls.png" alt="" class="file-img"
									     v-if="item.extension == 'xls'">
									<img
											src="@/static/assets/xlsx.png"
											alt=""
											class="file-img"
											v-if="item.extension == 'xlsx'">
									<img src="@/static/assets/csv.png" alt="" class="file-img"
									     v-if="item.extension == 'csv'">
									<img src="@/static/assets/ppt.png" alt="" class="file-img"
									     v-if="item.extension == 'ppt'">
									<img
											src="@/static/assets/pptx.png"
											alt=""
											class="file-img"
											v-if="item.extension == 'pptx'">
									<img src="@/static/assets/txt.png" alt="" class="file-img"
									     v-if="item.extension == 'txt'">
									<img src="@/static/assets/pdf.png" alt="" class="file-img"
									     v-if="item.extension == 'pdf'">
									<img
											src="@/static/assets/xmind.png"
											alt=""
											class="file-img"
											v-if="item.extension == 'xmind'">
									<div class="file-title">{{item.file_name}}</div>
								</template>
							</van-cell>
						</van-list>

					</van-pull-refresh>
					<van-pull-refresh v-if="fileType == 3"
					                  v-model="isVideoLoading"
					                  success-text="刷新成功"
					                  @refresh="onRefresh">
						<van-list
								v-model="videoLoading"
								:finished="videoFinished"
								finished-text="没有更多了"
								@load="loadingItem">
							<van-cell
									v-for="(item,index) in videoList"
									:key="index">
								<template slot="title">
									<view style="height: 24px;padding: 10px 0;">
										<img src="@/static/assets/video.png"
										     @click="videoPreview(item.local_path,item)" alt=""
										     class="file-img"
										>
										<span class="file-title"
										      style="color: #323233;font-size: 14px;width: 100%;max-width: calc(100% - 60px);"
										      @click="videoPreview(item.local_path, item)">{{item.file_name}}</span>
										<!--										<van-icon-->
										<!--												:name="includeId(chooseMsgH5OrFileArr, item.id) != -1 ? 'passed' : 'circle'"-->
										<!--												class="icon-passed"-->
										<!--												:class="includeId(chooseMsgH5OrFileArr, item.id) != -1?'active2' :''"-->
										<!--												@click="chooseMsg(item)"-->
										<!--												style="margin-top: 10px;margin-right: 5px;font-size: 22px;"/>-->
									</view>
								</template>
							</van-cell>
						</van-list>
					</van-pull-refresh>
					<!--				</van-tab>-->
					<!--			</van-tabs>-->
				</div>
			</div>
		</div>
		<div v-if="!showList" style="padding: 20px;">
			<img src="@/static/assets/site.png" alt="" style="max-width: 300px; width: 100%;">
			<h2>欢迎使用云美来智慧营销系统</h2>
		</div>
		<Footer v-if="is_show_copyright == 1" style="position: fixed;bottom: 0;left: 0;right: 0;"></Footer>
	</div>
</template>

<script>
	import {Toast} from 'vant';
	import Group from "@/components/Group";
	import Footer from "../../components/footer/footer.vue";

	export default {
		name      : 'index',
		props     : {
			msg: String
		},
		components: {
			Group,Footer
		},
		inject    : ['initPage', 'getParameter', 'getCopyright'],
		data () {
			return {
				wxReady         : false,
				showList        : false,
				corpid          : '',//企业微信的corpid，必须与当前登录的企业一致
				agentId         : '',//企业微信的应用id
				chatId          : '',//群id
				agentkey        : '',
				url             : '',
				ticketData      : {},
				agentData       : {},
				uid             : '',
				ids             : '',
				fileType        : 5,
				activeNames     : ['1'],
				groupName       : '',//获取的分组名称
				group_id        : '',//获取的分组id
				showPicker      : false,//分组弹窗的显示与隐藏
				inputValue      : '',//搜索框内容
				resultId        : '',//搜索框搜索的素材id
				groupArr        : [],//分组
				page            : 1,//页数
				pageSize        : 15,//每页默认15条
				count           : '',//所有素材总数量
				//文件
				isFileLoading   : false,//文件下拉刷新的Loading
				fileList        : [],
				fileLoading     : false,//文件的加载组件
				fileFinished    : false,
				//h5
				ishFiveLoading  : false,//h5下拉刷新的Loading
				hFiveList       : [],
				hFiveLoading    : false,//h5的加载组件
				noMorehFive     : false,//图片没有更多的显示与隐藏
				previewTitle    : '',   // 预览NavBar标题
				//视频
				isVideoLoading  : false,//视频下拉刷新的Loading
				videoList       : [],
				videoLoading    : false,//视频的加载组件
				videoFinished   : false,
				videourl        : '',
				showVideoPreview: false,
				// 图片
				noMorePic       : false,
				picList         : [],
				picLoading      : false,
				isPicLoading    : false,
				images          : [],
				height          : {
					height: ''
				},//图片的高度
				show            : false,
				previewUrl      : '',

				attachId   : '',
				statisticId: '',
				interval   : '',
				is_show_copyright : localStorage.getItem('is_show_copyright'),//是否展示底部版權
			};
		},
		methods   : {
			h5Preview (url) {
				if (url.indexOf(this.$store.state.h5Url) > -1) {
					window.open(url + '&history_url=' + encodeURIComponent(encodeURIComponent(window.location.href)))
				} else {
					window.open(url)
				}
			},
			initData () {
				this.page = 1

				this.fileList = []
				this.hFiveList = []
				this.videoList = []
				this.picList = []

				this.fileFinished = false
				this.noMorehFive = false
				this.videoFinished = false
				this.noMorePic = false

				this.ishFiveLoading = false
				this.isFileLoading = false
				this.isPicLoading = false
				this.isVideoLoading = false

				this.hFiveLoading = true
				this.fileLoading = true
				this.picLoading = true
				this.videoLoading = true
			},
			// 获取分组
			async getGroupList () {
				const {data: res} = await this.axios.post("chat-message/group", {
					uid: this.uid,
					ids: this.ids
				});
				if (res.error != 0) {
					Toast.fail(res.error_msg);
				} else {
					this.groupArr = res.data.group;
					let g = {
						key         : "",
						value       : "0",
						parent_id   : null,
						title       : "所有",
						sort        : 0,
						is_not_group: 0,
						scopedSlots : {
							title: "custom"
						},
						children    : []
					}
					this.groupArr.splice(0, 0, g)
				}
			},
			//分组下的确定、取消
			handlerOk () {
				this.group_id = this.$store.state.group_id
				this.groupName = this.$store.state.groupName
				this.showPicker = false
				this.page = 1
				this.getList()
			},
			handlerCancle () {
				this.$store.commit("setGroupId", this.group_id);
				this.$store.commit("setGroupName", this.groupName);
				this.showPicker = false
			},
			clearGroup () {
				this.$store.commit("setGroupId", '');
				this.$store.commit("setGroupName", '');
			},
			queryName () {
				this.page = 1
				this.getList()
			},
			//获取素材列表
			async getList () {
				if (this.fileType == 1) {
					this.picLoading = true
				}
				const {data: res} = await this.axios.post("chat-message/send-display", {
					uid     : this.uid,
					ids     : this.ids,
					agent_id: this.agentId,
					chat_id : this.chatId,
					name    : this.inputValue,
					group_id: this.group_id,
					page    : this.page,
					pageSize: this.pageSize,
				});
				if (res.error != 0) {
					Toast.fail(res.error_msg);
				} else {
					this.count = parseInt(res.data.count)
					this.fileType = res.data.file_type
					if (this.fileType == 5) {
						//文件
						if (this.page == 1) {
							this.fileList = res.data.attachment
						} else {
							this.fileList = this.fileList.concat(res.data.attachment)
						}
						// 数据全部加载完成
						if (this.fileList.length >= this.count) {
							this.fileFinished = true
						}
						this.fileLoading = false
						this.isFileLoading = false
					} else if (this.fileType == 4) {
						// 图文
						if (this.page == 1) {
							this.hFiveList = res.data.attachment
						} else {
							this.hFiveList = this.hFiveList.concat(res.data.attachment)
						}
						// 数据全部加载完成
						if (this.hFiveList.length >= this.count) {
							this.noMorehFive = true
						}
						this.hFiveLoading = false
						this.ishFiveLoading = false
					} else if (this.fileType == 1) {
						if (this.page == 1) {
							this.picList = res.data.attachment
						} else {
							this.picList = this.picList.concat(res.data.attachment)
						}
						this.images = []
						this.picLoading = false
						this.isPicLoading = false
						for (let i = 0; i < this.picList.length; i++) {
							this.images.push(this.picList[i].local_path)
						}
						//动态让图片高度等于宽度
						let width = (window.innerWidth - 46) / 3
						this.height.height = parseInt(width) + 'px'
						if(this.picList.length == 1) {
							this.previewPic(this.picList[0].local_path || this.picList[0].s_local_path, this.picList[0].id)
						}
					} else if (this.fileType == 3) {
						//视频
						if (this.page == 1) {
							this.videoList = res.data.attachment
						} else {
							this.videoList = this.videoList.concat(res.data.attachment)
						}
						this.isVideoLoading = false
						// 加载状态结束
						this.videoLoading = false
						if (this.videoList.length >= this.count) {
							// 数据全部加载完成
							this.videoFinished = true;
						}
						if(this.videoList.length == 1) {
							this.videoPreview(this.videoList[0].local_path, this.videoList[0])
						}
					}
				}
			},
			//预览图片
			previewPic (url, id) {
				this.previewUrl = url
				this.attachId = id
				this.getStatisticId()
				this.show = true
			},
			cancelShow () {
				this.show = false
				this.previewUrl = ''
				this.attachId = ''
				clearInterval(this.interval)
			},
			videoPreview (url, item) {
				this.previewTitle = item.file_name
				this.attachId = item.id
				this.getStatisticId()
				this.videourl = url
				this.showVideoPreview = true
			},
			cancelVideo () {
				this.showVideoPreview = false
				this.previewTitle = ''
				clearInterval(this.interval)
				this.attachId = ''
				this.videourl = ''
			},
			async getStatisticId () {
				let params = {
					attach_id: this.attachId,
				}
				if (localStorage.getItem("user_id")) {
					// 外部联系人
					// params['user'] = localStorage.getItem('user_id')
					// params['user_type'] = 1
				} else if (localStorage.getItem("external_id")) {
					// 外部联系人
					params['user'] = localStorage.getItem('external_id')
					params['user_type'] = 2
				} else if (localStorage.getItem("openid")) {
					// 普通用户
					params['user'] = localStorage.getItem('openid')
					params['user_type'] = 3
				}
				// else if (localStorage.getItem("user_id")) {
				// 	// 企业成员 暂不记录
				// 	params.user = localStorage.getItem('user_id')
				// 	params.user_type = 1
				// }
				const {data: res} = await this.axios.post("image-text/get-statistic-id", params);
				if (res.error != 0) {
					// Toast.fail(res.error_msg);
				} else {
					let that = this
					that.statisticId = res.data.statistic_id
					if (this.getParameter("agent_id") !== null) {
						that.interval = setInterval(() => {
							that.send()
						}, 300)
					}
				}
			},
			send () {
				this.ws.websocketSend(
					JSON.stringify({
						channel: "pull",
						info   : {
							type        : 1,
							statistic_id: this.statisticId
						}
					})
				)
			},
			//下拉刷新
			onRefresh () {
				this.initData()
				this.getList()
			},
			openFile (url) {
				window.open(url)
			},
			loadingItem () {
				if (this.count == '') {
					return false
				}
				if (this.fileType == 5) {
					this.fileLoading = false
					if (this.fileList.length < this.count) {
						this.page++
						let a = this.count / this.pageSize
						let b = ''
						if (this.count % this.pageSize !== 0) {
							b = parseInt(a) + 1
						} else {
							b = a
						}
						if (this.page <= b) {
							this.getList()
						}
					} else {
						// 加载状态结束
						this.fileLoading = false;
						// 数据全部加载完成
						this.fileFinished = true;
					}
				} else if (this.fileType == 4) {
					this.hFiveLoading = false
					this.ishFiveLoading = false
					if (this.hFiveList.length < this.count) {
						this.page++
						let a = this.count / this.pageSize
						let b = ''
						if (this.count % this.pageSize !== 0) {
							b = parseInt(a) + 1
						} else {
							b = a
						}
						if (this.page <= b) {
							this.getList()
						}
					} else {
						this.noMorehFive = true
						this.hFiveLoading = false
						this.ishFiveLoading = true
					}
				} else if (this.fileType == 3) {
					this.videoLoading = true
					if (this.videoList.length == 0) {
						this.getList()
					} else if (this.videoList.length < this.count) {
						this.page++
						let a = this.count / this.pageSize
						let b = ''
						if (this.count % this.pageSize !== 0) {
							b = parseInt(a) + 1
						} else {
							b = a
						}
						if (this.page <= b) {
							this.getList()
						}
					}
				}
			},
			//监听页面滑动，要不要加载
			handleScroll () {
				//变量scrollTop是滚动条滚动时，距离顶部的距离
				var scrollTop = document.getElementsByClassName("metarial-list")[0].scrollTop;
				//变量windowHeight是可视区的高度
				var windowHeight = document.getElementsByClassName("metarial-list")[0].clientHeight;
				//变量scrollHeight是滚动条的总高度
				var scrollHeight = document.getElementsByClassName("metarial-list")[0].scrollHeight;
				//滚动条到底部的条件
				if (scrollTop + windowHeight == scrollHeight) {
					// window.console.log('触发啦')
					if (this.fileType == 1) {
						this.picLoading = true
						if (this.picList.length < this.count) {
							this.page++
							let a = this.count / this.pageSize
							let b = ''
							if (this.count % this.pageSize !== 0) {
								b = parseInt(a) + 1
							} else {
								b = a
							}
							if (this.page <= b) {
								this.getList()
							}
						} else {
							this.picLoading = false
							this.noMorePic = true
						}
					}
				}
			},
			//一进入页面请求该方法获取应用的权限
			async limit () {
				this.url = window.location.href
				this.userId = localStorage.getItem('user_id') != null ? localStorage.getItem('user_id') : ''
				let agent_id = this.$route.query.agent_id
				localStorage.setItem('agent_id', agent_id)
				const {data: res} = await this.axios.post("chat-message/get-config", {
					url     : this.url,
					agent_id: agent_id
				});
				if (res.error != 0) {
					Toast.fail(res.error_msg);
				} else {
					this.uid = res.data.uid
					this.corpid = res.data.corpid
					localStorage.setItem('uid', this.uid)

					if (localStorage.getItem('uid')) {
						//获取底部技术支持
						this.getCopyright(this.refresh)
					}

					localStorage.setItem('corpid', this.corpid)
					this.getGroupList()
					this.getList()
				}
			},
			//重新渲染頁面
			refresh () {
				this.is_show_copyright = localStorage.getItem('is_show_copyright')
				this.$forceUpdate()
			},
		},
		onShow () {
			this.showList = true
			if (this.$route.query.ids) {
				this.ids = this.$route.query.ids
			}
			if (this.$route.query.uid) {
				this.uid = this.$route.query.uid

				localStorage.setItem('uid', this.uid)
				if (localStorage.getItem('uid')) {
					//获取底部技术支持
					this.getCopyright(this.refresh)
				}
			}
			if (this.$route.query.agent_id) {
				this.agentId = this.$route.query.agent_id
			}

			if (this.$route.query.chat_id) {
				this.chatId = this.$route.query.chat_id
			}

			if (this.$route.query.file_type) {
				this.fileType = this.$route.query.file_type
				let _this = this
				if(this.fileType == 1){
					uni.setNavigationBarTitle({
						title: '图片查看'
					});
				}
				if(this.fileType == 3){
					uni.setNavigationBarTitle({
						title: '视频查看'
					});
				}
				if (this.fileType == 1 || this.fileType == 3) {
					this.$store.dispatch('setWx', () => {
						_this.initPage(_this.limit)
					})
				} else {
					this.getGroupList()
					this.getList()
				}
				if (this.fileType == 1) {
					this.$nextTick(function () {
						window.document.getElementsByClassName("metarial-list")[0].addEventListener('scroll', _this.handleScroll);
					})
				}
			} else {
				this.getGroupList()
				this.getList()
			}
		},
		onLoad () {
		}
	}
</script>

<style>
	@import 'vant/lib/index.css'
</style>

<style scoped>
	/*.hello {*/
	/*	background: #F7F8FA;*/
	/*}*/
	body > iframe {
		position: fixed !important;
	}

	.van-tab__pane, .van-tab__pane-wrapper {
		padding: 0 10px;
		border-bottom: 1px solid #F2F2F2;
	}

	.van-cell__title {
		width: 100%;
		text-align: left;
	}

	.van-cell__value {
		flex: 0;
	}

	/deep/ .van-cell {
		margin: 0px;
	}

	/deep/ .van-tabs__content {
		padding-top: 44px;
	}

	/deep/ .van-tabs--line .van-tabs__wrap {
		position: fixed;
		left: 0;
		right: 0;
		z-index: 999;
	}

	.list-text {
		display: inline-block;
		width: 100%;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
		max-width: calc(100% - 20px);
		text-align: left;
	}

	.list-text2 {
		display: inline-block;
		width: 100%;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
		text-align: left;
		color: #969799;
		font-size: 12px;
		line-height: 18px;
	}

	/deep/ .van-grid-item__content {
		padding: 0;
	}

	.pic-text {
		position: absolute;
		bottom: -25px;
		left: 0;
		right: 0;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
		font-size: 14px;
		padding: 3px;
		background: #0F0F0F;
		color: #FFF;
		opacity: 0.3;
		line-height: 19px;
	}

	.file-text {
		font-size: 14px;
		width: 100%;
		margin: 5px 0;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
	}

	/deep/ .van-tabs {
		top: 98px;
		right: 0;
		bottom: 44px;
		left: 0;
		position: fixed;
		overflow-y: auto;
	}

	.active {
		border: 2px solid #1989FA;
	}

	.active2 {
		color: #1989FA !important;
	}

	/deep/ .van-grid-item {
		padding-right: 0px !important;
		flex-basis: 30.5233% !important;
		margin-right: 4.21505% !important;
		padding-bottom: 25px;
	}

	/deep/ .van-grid-item:nth-child(3n+3) {
		padding-right: 0 !important;
		flex-basis: 30.5233% !important;
		margin-right: 0px !important;
	}

	.slider-text-label {
		text-align: left;
		word-break: break-word;
		overflow: hidden;
		text-overflow: ellipsis;
		/*text-overflow: -o-ellipsis-lastline;*/
		line-clamp: 2;
		display: -webkit-box;
		-webkit-box-orient: vertical;
		-webkit-line-clamp: 2;
	}

	.img {
		max-width: 100%;
	}

	.img2 {
		width: 60px;
		height: 33px;
		margin-top: 3px;
	}

	.chooseGroupName {
		font-size: 14px;
		display: inline-block;
		max-width: calc(100% - 64px);
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
	}

	.file-img {
		width: 24px;
		float: left;
	}

	.file-title {
		float: left;
		margin-left: 15px;
		font-size: 16px;
		max-width: calc(100% - 39px);
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
		text-align: left;
	}

	.video-box {
		max-width: 100%;
		height: 0;
		padding-bottom: 100%;
		overflow: hidden;
		border: 1px solid #F2F2F2;
		line-height: 240px;
	}

	.video-img {
		max-width: 100%;
		max-height: 165.5px;
	}

	.video-text {
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
		font-size: 14px;
		padding: 5px;
		background: #0F0F0F;
		color: #FFF;
		opacity: 0.3;
		line-height: 15px;
	}

	.play-video {
		font-size: 40px;
		position: absolute;
		left: 50%;
		margin-left: -20px;
		margin-top: -20px;
		color: #646566;
	}

	.icon-passed {
		position: absolute;
		top: 5px;
		right: 5px;
		font-size: 22px;
		color: #C8C9CC;
	}

	/deep/ .van-image-preview__cover {
		width: 100%;
	}

	.web-back {
		top: 0;
		right: 0;
		left: 0;
		height: 44px;
		z-index: 1004;
		position: fixed;
	}

	.web-view {
		top: 43px;
		right: 0;
		bottom: 0;
		left: 0;
		position: fixed;
	}

	/deep/ .uni-navbar__header {
		background: rgba(0, 0, 0, .1) !important;
	}

	/deep/ .uni-navbar__header-container {
		width: calc(100% - 160px);
	}

	.preview-box {
		top: 0;
		right: 0;
		bottom: 0;
		left: 0;
		position: fixed;
		z-index: 9999999;
		background-color: #000000;
	}

	.swiper-box {
		height: 100%;
	}

	.swiper-item {
		height: 100%;
		display: flex;
		align-items: center;
		justify-content: center;
	}
</style>
