<template>

</template>

<script>
//  import axios from 'axios'

  export default {
    name: 'dragAndDropMixin',
    data() {
      return {
        /* create map of background board tiles, that indicate the position of the set dames.
        *  an static method was chosen for approachability to the code */
        borderTiles: [],
        tiles: [],
        remainingDames: []
      }
    },
    methods: {
      /*handle drag and dropping the dames manually*/
      allowDrop(evt) {
//        console.log('allowDrop evt: ', evt, evt.target);
        evt.target.tagName === "SPAN" && !evt.target.classList.contains('tile--threatened') && evt.preventDefault()
      },
      dragLeave(evt) {
        evt.target.parentNode.removeAttribute('drop-active')
        evt.dataTransfer.setData("text", evt.target.id)
      },
      dragEnter(evt) {
//        console.log('allowDrop evt: ', evt.target);
        evt.target.parentNode.setAttribute('drop-active', true)
      },
      dragStart(evt) {
        console.log('dragstart evt: ', evt/*, evt.srcElement, 'evt.target.id: '*/, evt.target.id)
        evt.dataTransfer.setData("text", evt.target.id)
//        evt.dataTransfer.setDragImage(document.querySelector('#dame_0'), 999, 999);
      },
      /* executes whenever the dame image is dropped on a chess tile or a dame-pool-tile
      @params evt (Event): the drop type event when a dragged element is release by mouseclick
      @params rowIndex (Number): maps the tile to the row in the tiles Array in the data property
      @params tileIndex (Number): maps the tile to the index within a row in the tiles Array in the data property
      @params remainingDamesIndex (Number): gives the tile index in the dame pool
      * */
      drop(evt, rowIndex, tileIndex, remainingDamesIndex) {

//        remove the "droppable to" tile to remove highlighting on the tile
        evt.target.removeAttribute('drop-active')
        evt.target.parentNode.removeAttribute('drop-active')
        evt.preventDefault() //actually do drop the dame img on the tile

        var data = evt.dataTransfer.getData("text")
        var draggedElement = document.getElementById(data)
        if(!draggedElement) return ;
        var dameNumber = (draggedElement.dataset['dameNumber']) ? parseInt(draggedElement.dataset['dameNumber']) : -1
        var targetTile = null
        if (rowIndex !== undefined && tileIndex !== undefined) {
          targetTile = this.tiles[rowIndex][tileIndex]
          targetTile.occupiedByDame = dameNumber
          console.log('this.tiles: ', this.tiles)
        }

        //@ToDo reset occupiedByDame on the field tile, if returned to dame-pool-tile
        console.log('this.remainingDames: ', this.remainingDames, 'valid dame index '+(remainingDamesIndex !== undefined))
        if (remainingDamesIndex !== undefined) {
          this.remainingDames[remainingDamesIndex].occupiedByDame = parseInt(dameNumber)
        }

        /*its a dame pool tile, so set remove its*/
//        evt.srcElement.parentNode.id.indexOf('dame-pool-tile') > -1
        console.log('draggedElement: ', evt, targetTile, dameNumber ,draggedElement)
        evt.target.appendChild(draggedElement)
      }
    }
  }
        //@ToDo: comment the code
</script>

<style lang="stylus">
</style>

