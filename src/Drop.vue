<template>
	<component :is="tag"
		@dragenter="emitEvent('dragenter', $event)"
		@dragleave="emitEvent('dragleave', $event)"
		@dragover.prevent="emitEvent('dragover', $event)"
		@drop.prevent="emitEvent('drop', $event)"
	>
		<slot :transfer-data="scopedData"></slot>
	</component>
</template>

<script>
	import transferDataStore from './transferDataStore';

	const insideElements = new Set();

	export default {
		data() {
			return { transferData: undefined, isDraggingOver: false };
		},
		props: {
			tag: { type: String, default: 'div' },
		},
		computed: {
			scopedData() {
				return this.isDraggingOver && this.transferData;
			},
		},
		methods: {
			emitEvent(name, nativeEvent) {
				this.transferData = transferDataStore.data;
				this.$emit(name, this.transferData, nativeEvent);

				/**
				 * After emitting the event, we need to determine if we're still
				 * dragging inside this Drop. We keep a Set of all elements that we've
				 * dragged into, then clear the data if that set is empty.
				 */

				// Add to the set on dragenter.
				if (name === 'dragenter') {
					if (insideElements.size || nativeEvent.target === this.$el) {
						insideElements.add(nativeEvent.target);
					}
				}

				// Remove from the set on dragleave.
				if (name === 'dragleave') {
					insideElements.delete(nativeEvent.target);
				}

				// A drop resets everything.
				if (name === 'drop') {
					insideElements.clear();
				}

				// Finally, since Vue can't react to Set changes, set a flag indicating drag status.
				this.isDraggingOver = Boolean(insideElements.size);
			},
		},
	};
</script>
