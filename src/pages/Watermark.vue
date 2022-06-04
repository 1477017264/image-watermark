<script setup>
	import {
		computed,
		nextTick,
		reactive,
		ref
	} from 'vue'
	import html2canvas from 'html2canvas'

	// 图片上传
	const uploadRef = ref(null)
	const imgSrc = ref('')
	let imgType = ''

	function handleChange(file) {
		imgSrc.value = URL.createObjectURL(file.raw)
		imgType = file.raw.type
	}

	function handleExceed(files) {
		URL.revokeObjectURL(imgSrc.value) // 释放图片对象
		// 替换上一个文件
		uploadRef.value.clearFiles()
		uploadRef.value.handleStart(files[0])
	}

	// 水印配置
	const config = reactive({
		content: '水印文字',
		vertical: false,
		spacing: 0,
		size: 24,
		color: '#888888',
		opacity: 0.8,
		rotate: -30,
		numX: 6,
		numY: 6,
		gapX: 40,
		gapY: 100,
		offsetX: 0,
		offsetY: 30,
	})
	const gapXVal = computed(() => `${config.gapX}px`)
	const gapYVal = computed(() => `${config.gapY}px`)
	const offsetXVal = computed(() => `${config.offsetX}px`)
	const offsetYVal = computed(() => `${config.offsetY}px`)
	const rotateVal = computed(() => `${config.rotate}deg`)
	const sizeVal = computed(() => `${config.size}px`)
	const colStyle = computed(() => ({
		width: config.vertical ? `${config.size}px` : '',
		letterSpacing: config.vertical ? '' : `${config.spacing}px`,
		lineHeight: config.vertical ? `calc(${config.size + config.spacing}px)` : '',
	}))

	// 生成
	const wrapperWidth = 360 // 预览宽度
	const previewRef = ref(null)
	const imgRef = ref(null)
	const isGenerate = ref(false)
	const watermarkScale = ref(1)
	const previewStyle = computed(() => ({
		display: isGenerate.value ? 'inline-block' : 'block',
	}))
	const imgStyle = computed(() => ({
		width: isGenerate.value ? 'auto' : '100%',
	}))
	const isShowResult = ref(false)
	const resultSrc = ref('')
	async function generate() {
		if (!imgSrc.value) {
			ElMessage('请选择图片')
			return
		}
		isGenerate.value = true
		await nextTick()
		watermarkScale.value = imgRef.value.width / wrapperWidth
		await nextTick()
		try {
			const canvas = await html2canvas(previewRef.value)
			resultSrc.value = canvas.toDataURL(imgType)
			isShowResult.value = true
		} catch (error) {
			ElMessage.error('生成出错')
		}
		// 重置回预览状态
		watermarkScale.value = 1
		isGenerate.value = false
	}
</script>

<template>
	<el-row :gutter="10">
		<el-col :xs="24" :sm="8">
			<el-form :model="config" label-width="80px" class="form">
				<el-form-item label="上传图片">
					<el-upload ref="uploadRef" action="" :show-file-list="false" :limit="1" :auto-upload="false"
						:on-change="handleChange" :on-exceed="handleExceed">
						<template #trigger>
							<el-button type="primary">选择文件</el-button>
						</template>
						<template #default></template>
					</el-upload>
				</el-form-item>
				<el-form-item label="水印内容">
					<el-input placeholder="水印内容" v-model="config.content" />
				</el-form-item>
				<el-form-item label="文字方向">
					<el-radio-group v-model="config.vertical">
						<el-radio-button :label="false">横</el-radio-button>
						<el-radio-button :label="true">竖</el-radio-button>
					</el-radio-group>
				</el-form-item>
				<el-form-item label="文字间距">
					<el-input-number :min="0" v-model="config.spacing" />
				</el-form-item>
				<el-form-item label="文字大小">
					<el-input-number :min="0" :step="2" v-model="config.size" />
				</el-form-item>
				<el-form-item label="文字颜色">
					<el-color-picker v-model="config.color" />
				</el-form-item>
				<el-form-item label="水印明度">
					<el-slider :min="0" :max="1" :step="0.1" v-model="config.opacity"></el-slider>
				</el-form-item>
				<el-form-item label="水印角度">
					<el-input-number :min="-90" :max="90" :step="10" v-model="config.rotate" />
				</el-form-item>
				<el-form-item label="X 轴数量">
					<el-input-number :min="1" v-model="config.numX" />
				</el-form-item>
				<el-form-item label="Y 轴数量">
					<el-input-number :min="1" v-model="config.numY" />
				</el-form-item>
				<el-form-item label="X 轴间隔">
					<el-input-number :step="10" v-model="config.gapX" />
				</el-form-item>
				<el-form-item label="Y 轴间隔">
					<el-input-number :step="10" v-model="config.gapY" />
				</el-form-item>
				<el-form-item label="X 轴偏移">
					<el-input-number :step="10" v-model="config.offsetX" />
				</el-form-item>
				<el-form-item label="Y 轴偏移">
					<el-input-number :step="10" v-model="config.offsetY" />
				</el-form-item>
				<el-form-item>
					<el-button type="primary" :loading="isGenerate" @click="generate">
						立即生成
					</el-button>
				</el-form-item>
			</el-form>
		</el-col>
		<el-col :xs="24" :sm="16">
			<div v-if="imgSrc" class="wrapper">
				<div ref="previewRef" class="preview" :style="previewStyle">
					<img ref="imgRef" :src="imgSrc" class="img" :style="imgStyle" @load="handleLoad" />
					<div class="watermark">
						<div v-for="y in config.numY" class="row">
							<div v-for="x in config.numX" class="col" :style="colStyle">
								{{ config.content }}
							</div>
						</div>
					</div>
				</div>
			</div>
		</el-col>
	</el-row>

	<el-dialog title="右击(移动端长按)图片保存" center v-model="isShowResult">
		<img :src="resultSrc" class="result" />
	</el-dialog>
	<div class="footer">
		<span>图片压缩等更多工具</span>
		<a href="https://dadio.cc/website/" target="_blank">点击前往</a>
	</div>
</template>

<style scoped>
	.el-dialog--center {
		text-align: center;
		width: 340px !important;
	}

	.el-form-item--default {
		margin: 10px 0;
	}

	.el-upload.el-upload--text button {
		width: 100%;
	}

	.el-form-item__content:nth-last-child() {
		margin-left: 0 !important;
	}

	.el-form-item__content .el-button {
		width: 100%;
	}

	.is-guttered {
		display: block;
		display: flex;
		min-height: 1px;
		flex-direction: column;
		flex-wrap: nowrap;
		align-content: center;
		justify-content: center;
		align-items: center;
	}

	.is-guttered .form {
		max-width: 400px;
		display: flex;
		flex-direction: column;
		flex-wrap: nowrap;
		justify-content: center;
		align-content: center;
	}

	.el-row {
		display: flex;
		flex-direction: row;
		flex-wrap: nowrap;
		align-content: center;
		justify-content: center;
		align-items: center;
	}

	.el-color-picker {
		width: 100%;
	}

	.wrapper {
		overflow: hidden;
		width: v-bind(wrapperWidth + 'px');
		overflow: hidden;
		border-radius: 8px;
		box-shadow: 0 0 5px 2px rgba(0, 0, 0, 0.15);
	}

	.preview {
		position: relative;
		overflow: hidden;

	}

	.img {
		display: block;
		width: 100%;
	}

	.watermark {
		position: absolute;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		z-index: 1;
		font-size: v-bind(sizeVal);
		color: v-bind('config.color');
		opacity: v-bind('config.opacity');
		transform: scale(v-bind(watermarkScale)) translate(v-bind(offsetXVal), v-bind(offsetYVal));
		transform-origin: 0 0;
	}

	.row {
		white-space: nowrap;
	}

	.col {
		display: inline-block;
		white-space: normal;
		margin-right: v-bind(gapXVal);
		margin-bottom: v-bind(gapYVal);
		line-height: 1;
		overflow-wrap: break-word;
		text-align: center;
		transform: rotate(v-bind(rotateVal));
	}

	.result {
		width: 100%;
		border-radius: 8px;
		box-shadow: 0 0 10px 2px rgb(0 0 0 / 20%);
	}

	.el-radio-button__original-radio {
		width: 100%;
	}

	.is-guttered:nth-child(1) {
		margin: auto 40px;
	}

	.footer {
		width: 100%;
		margin: 15px auto 10px;
		display: flex;
		flex-direction: row;
		flex-wrap: nowrap;
		align-content: center;
		justify-content: center;
		align-items: center;
		background: rgba(0, 0, 0, 0.05);
		height: 44px;
		border-radius: 8px;
		position: relative;
		font-weight: bold;
		color: rgba(0, 0, 0, 0.7);
		font-family: sans-serif;
	}

	.footer a {
		font-size: 15px;
		position: absolute;
		background: rgb(58 142 230);
		transition: .3s;
		height: 36px;
		right: 4px;
		border-radius: 6px;
		width: 90px;
		display: flex;
		justify-content: center;
		align-items: center;
		text-decoration: none;
		color: #fff;
	}

	.footer a:hover {
		background: rgb(102 177 255);
	}

	@media screen and (max-width:420px) {
		.el-row {
			display: flex;
			flex-wrap: nowrap;
			position: relative;
			box-sizing: border-box;
			justify-content: center;
			align-items: center;
			align-content: center;
			flex-direction: column;
		}

		.is-guttered:nth-child(1) {
			margin: auto 0;
		}

		.wrapper {
			margin: 10px 0;
			overflow: hidden;
		}

		#app {
			margin: auto;
			width: 100%;
		}
	}
</style>
