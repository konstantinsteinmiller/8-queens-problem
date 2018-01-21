<template>

</template>

<script>
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
        /* if tile is not threathened allow the drop */
        evt.target.tagName === "SPAN" && !evt.target.classList.contains('tile--threatened') && evt.preventDefault()
//        console.log('allowDrop evt: ', evt, evt.target);
      },
      dragLeave(evt) {
        /* remove the mark for the red threat background */
        evt.target.parentNode.removeAttribute('drop-active')
        evt.dataTransfer.setData("text", evt.target.id)
      },
      dragEnter(evt) {
        /* add the mark for the possible red threat background */
        evt.target.parentNode.setAttribute('drop-active', true)
//        console.log('allowDrop evt: ', evt.target);
      },
      dragStart(evt) {
        evt.dataTransfer.setData("text", evt.target.id)
//        console.log('dragstart evt: ', evt/*, evt.srcElement, 'evt.target.id: '*/, evt.target.id)
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
//          console.log('this.tiles: ', this.tiles)
        }

//        console.log('this.remainingDames: ', this.remainingDames, 'valid dame index '+(remainingDamesIndex !== undefined))
        if (remainingDamesIndex !== undefined) {
          this.remainingDames[remainingDamesIndex].occupiedByDame = parseInt(dameNumber)
        }

        /*its a dame pool tile, so set remove its*/
        evt.target.appendChild(draggedElement)
      }
    }
  }
</script>

<style lang="stylus">
</style>

