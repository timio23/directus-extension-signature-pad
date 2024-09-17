<template>
	<div>
		<input 
			type="hidden"
			v-model="signatureSVG"
		/>
		<div v-if="isLocked" class="signature-container">
			<img :src="value" width="296" height="96" />
		</div>
		<VueSignaturePad
			width="300px"
			height="100px"
			:customStyle="{
				'border': 'var(--input-border-width) solid var(--input-border-normal)',
				'border-radius': 'var(---input-border-radius)',
				'margin-bottom': '20px',
				'display': signatureDisplay,
			}"
			ref="signaturePad"
			:options="{ onBegin, onEnd }"
		/>
		<v-button v-if="!disabled" secondary @click="clearCanvas">Clear</v-button>
	</div>
</template>

<script>
import  { VueSignaturePad } from 'vue-signature-pad';
import { ref, watch } from 'vue';
export default {
	components: {
		VueSignaturePad
	},
	props: {
		value: String,
		disabled: Boolean,
	},
	emits: ['input'],
	setup(props, { emit }) {
		const signaturePad = ref(null);
		const signatureSVG = ref(null);
		const isLocked = ref(false);
		const signatureDisplay = ref('block');

		watch(
			[
				() => props.value,
			],
			() => {
				if(signatureSVG.value == null && props.value != null){
					signaturePad.value.fromDataURL(props.value);
					signatureSVG.value = props.value;
					signaturePad.value.lockSignaturePad();
					isLocked.value = true;
					signatureDisplay.value = 'none';
				}
			},
		);

		function clearCanvas() {
			if(props.disabled) return;
			signaturePad.value.openSignaturePad();
			signaturePad.value.clearSignature();
			signatureSVG.value = null;
			isLocked.value = false;
			signatureDisplay.value = 'block';
			emit('input', null);
			return;
		}

		function onEnd(){
			const { isEmpty, data } = signaturePad.value.saveSignature("image/svg+xml");
			signatureSVG.value = data;
			emit('input', data);
			return;
		}

		function onBegin(){
			return;
		}

		return { signatureSVG, signaturePad, clearCanvas, onBegin, onEnd, isLocked, signatureDisplay };
	},
};
</script>
<style scoped>
	.signature-container {
		width:300px;
		border: var(--border-width) solid var(--border-normal);
		border: var(--input-border-width) solid var(--input-border-normal);
		border-radius: var(--border-radius);
		border-radius: var(--input-border-radius);
		margin-bottom: 20px;
	}

	.signature-container img {
		display: block;
	}
</style>