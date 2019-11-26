<template>
	<component :is="tag"
		:draggable="draggable"
		@drag="reEmit"
		@dragstart="dragStart"
		@dragenter="dragEnter"
		@dragleave="reEmit"
		@dragend="dragEnd"
		v-on="$listeners"
	>
		<slot :transfer-data="scopedData"></slot>
		<template v-if="$slots.image || $scopedSlots.image">
			<div v-if="hideImageHtml" :style="hideImageStyle">
				<slot name="image" :transfer-data="scopedData"></slot>
			</div>
			<slot v-else name="image" :transfer-data="scopedData"></slot>
		</template>
	</component>
</template>

<script>
	import transferDataStore from './transferDataStore';

	export default {
		props: {
			draggable: {
				type: Boolean,
				default: true,
			},
			transferData: {},
			dropEffect: {
				validator: v => ['copy', 'move', 'link', 'none'].indexOf(v) >= 0,
			},
			effectAllowed: {
				validator: v => ['none', 'copy', 'copyLink', 'copyMove', 'link', 'linkMove', 'move', 'all', 'uninitialized'].indexOf(v) >= 0,
			},
			image: String,
			imageXOffset: {
				type: Number,
				default: 0,
			},
			imageYOffset: {
				type: Number,
				default: 0,
			},
			hideImageHtml: {
				type: Boolean,
				default: true,
			},
			tag: {
				type: String,
				default: 'div',
			},
		},

		data() {
			return {
				dragging: false,
			};
		},

		computed: {
			scopedData() {
				return this.dragging && this.transferData;
			},
			hideImageStyle() {
				return {
					position: 'fixed',
					top: '-1000px',
				};
			},
		},

		destroyed() {
			if (transferDataStore.drag === this) {
				this.resetStore();
			}
		},

		methods: {
			dragEnter(e) {
				if (this.dropEffect) {
					e.dataTransfer.dropEffect = this.dropEffect;
				}
				this.reEmit(e);
			},

			dragStart(e) {
				// Set the allowed effects
				if (this.effectAllowed) {
					e.dataTransfer.effectAllowed = this.effectAllowed;
				}

				// Set the drag image
				if (this.image || this.$slots.image) {
					let image;
					if (this.image) {
						image = new Image();
						image.src = this.image;
					} else if (this.$slots.image) {
						image = this.$slots.image[0].elm;
					}
					if (e.dataTransfer.setDragImage) {
						e.dataTransfer.setDragImage(image, this.imageXOffset, this.imageYOffset);
					}
				}

				transferDataStore.drag = this;

				// Set the transfer data
				if (this.transferData !== null) {
					transferDataStore.data = this.transferData;
					// Set a dummy string for the real transfer data. Not actually used
					// for anything, but necessary for browser compatibility.
					//
					// TODO: Maybe this should be the actual data serialized. But since
					// it's not actually used for anything it seems like a waste of CPU.
					e.dataTransfer.setData('text', '');
				}

				// Indicate that we're dragging.
				this.dragging = true;

				this.reEmit(e);
			},

			dragEnd(e) {
				this.reEmit(e);
				this.resetStore();
				this.dragging = false;
			},

			reEmit(e) {
				this.$emit(e.type, this.transferData, this, e);
			},

			resetStore() {
				transferDataStore.drag = null;
				transferDataStore.data = null;
			},
		},
	};
</script>
