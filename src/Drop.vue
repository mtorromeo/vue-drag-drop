<template>
	<component
		:is="tag"
		:class="{[hoverClass]: isDraggingOver}"
		@dragenter="dragEnter"
		@dragleave="dragLeave"
		@dragover.prevent="reEmit"
		@drop.prevent="reEmit"
	>
		<slot :transfer-data="scopedData"></slot>
	</component>
</template>

<script>
	import transferDataStore from './transferDataStore';

	const insideElements = new Set();

	export default {
		props: {
			tag: {
				type: String,
				default: 'div',
			},
			hoverClass: {
				type: String,
				default: null,
			},
		},

		data() {
			return {
				transferData: null,
				isDraggingOver: false,
			};
		},

		computed: {
			scopedData() {
				return this.isDraggingOver && this.transferData;
			},
		},

		methods: {
			dragEnter(e) {
				this.reEmit(e);

				/**
				 * After emitting the event, we need to determine if we're still
				 * dragging inside this Drop. We keep a Set of all elements that we've
				 * dragged into, then clear the data if that set is empty.
				 */
				this.alterInside(elements => {
					elements.add(e.target);
					if (transferDataStore.drag !== null) {
						transferDataStore.drag.$once('dragend', () => {
							this.alterInside(elements => elements.clear());
						});
					}
				});
			},

			dragLeave(e) {
				this.reEmit(e);
				this.alterInside(elements => elements.delete(e.target));
			},

			reEmit(e) {
				this.transferData = transferDataStore.data;
				this.$emit(e.type, this.transferData, this, e);
			},

			alterInside(f) {
				f(insideElements);
				this.isDraggingOver = Boolean(insideElements.size);
			},
		},

		watch: {
			isDraggingOver(over) {
				// non-native event that is triggered only when we are entering/leaving the dom subtree
				this.$emit(over ? 'enter' : 'leave', this.transferData, this);
			},
		},
	};
</script>
