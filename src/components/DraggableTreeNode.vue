<script>
import * as hp from 'helper-js'
import * as th from 'tree-helper'
import draggableHelper from 'draggable-helper'
import TreeNode from './TreeNode.vue'
import autoMoveDragPlaceHolder from './autoMoveDragPlaceHolder'
import * as ut from '../plugins/utils'

export default {
  extends: TreeNode,
  name: 'TreeNode',
  mounted() {
    if (this.isRoot || this.data.isDragPlaceHolder) {
      return
    }
    const {dplh} = this.store
    this.$watch('store.draggable', (draggable) => {
      if (ut.isPropTrue(draggable)) {
        const triggerEl = this.store.getTriggerEl ? this.store.getTriggerEl(this) : this.$el.querySelector('.tree-node-inner')
        this._draggableDestroy = draggableHelper(triggerEl, {
          // trigger el
          getEl: () => this.$el,
          minTranslate: 10,
          drag: (e, opt, store) => {
            // this store is not tree
            if (this.store.ondragstart && this.store.ondragstart(this.data, this, e, opt, store) === false) {
              return false
            }
            if (!isNodeDraggable(this.data)) {
              return false
            }
            dplh.innerStyle.height = store.el.offsetHeight + 'px'
            th.insertAfter(dplh, this.data)
            this.data.class += ' dragging'
            console.log('drag start');
          },
          moving: (e, opt, store) => {
            return autoMoveDragPlaceHolder.call(this, e, opt, store, this.store.trees)
          },
          drop: (e, opt, store) => {
            if (this.store.ondragend && this.store.ondragend(this.data, this, e, opt, store) === false) {
              // can't drop
            } else {
              th.insertAfter(this.data, dplh)
            }
            hp.arrayRemove(dplh.parent.children, dplh)
            this.data.class = this.data.class.replace(/(^| )dragging( |$)/g, ' ')
            console.log('drag end');
          },
        })
      } else {
        if (this._draggableDestroy) {
          this._draggableDestroy()
          delete this._draggableDestroy
        }
      }
    }, {immediate: true})
  },
}

function isNodeDraggable(node) {
  while (!node.hasOwnProperty('draggable') && node.parent) {
    node = node.parent
  }
  if (node.hasOwnProperty('draggable')) {
    return node.draggable
  } else {
    return true
  }
}
</script>
