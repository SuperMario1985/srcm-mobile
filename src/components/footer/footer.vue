<template>
	<view class="footer-background">
		<img :src="commonUrl + copyrightUrl" alt="" class="footer-img">
	</view>
</template>

<script>
	export default {
		props  : {},
		data () {
			return {
				commonUrl        : this.$store.state.commonUrl,
				copyrightUrl     : localStorage.getItem('copyright_url'),
			}
		},
		mounted () {
			this.copyrightUrl = localStorage.getItem('copyright_url')
			// this.copyrightUrl = '/upload/images/20210111/4edc985b84189933e110cf4aa900f68b1610346230.png'
		},
		methods: {},
	};
</script>

<style scoped>
	.footer-background {
		padding-bottom: 0.533rem /* 10/18.75 */;
		text-align: center;
	}

	.footer-img {
		width: 40%;
		margin: 0.533rem /* 10/18.75 */ 0;
	}
</style>
