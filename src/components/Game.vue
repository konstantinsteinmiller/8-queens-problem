<template>
  <div class="problem">
    <h1>{{ title }}</h1>
    <div class="game-map">

      <div class="game-map__field">
        <div v-for="(row, rowIndex) in tiles" :class="(rowIndex%2 === 0) ? 'row row--even' :'row row--odd'">
        <span v-for="(tile, tileIndex) in row"
              class="tile"
              :id="'tile_'+rowIndex+'_'+tileIndex"
              :class="[(tileIndex%2 === 0) ? 'tile--even' : 'tile--odd',
                      ((tileIndex%2 === 0 && rowIndex%2 === 0) || (tileIndex%2 === 1 && rowIndex%2 === 1)) ? 'tile--white' : 'tile--black']"
              @dragover="allowDrop($event)"
              @dragenter="dragEnter($event)"
              @dragleave="dragLeave($event)"
              @drop="drop($event, rowIndex, tileIndex)"
        >
          <img v-if="tile.occupied >= 0"
               draggable="true"
               @dragstart="drag($event)"
               src="/static/dame_transparent.png"
               :id="'dame_' + tile.occupiedByDame"
               :data-dame-number="tile.occupiedByDame"
               :class="[(tile.occupiedByDame >= 0) ? 'tile--occupied' : '', (tile.threatened) ? 'tile--threatened' : '']">

        </span>
        </div>
      </div>
      <div class="game-map__border">
        <div v-for="(borderRow, borderRowIndex) in borderTiles"
             class="row">
          <span v-for="(borderTile, borderTileIndex) in borderRow"
                class="tile"
                :class="(borderRowIndex > 0 && borderRowIndex <9 && borderTileIndex > 0 && borderTileIndex < 9) ? 'tile--background': ''">
            <span class="tile-label"><span v-if="borderTile.label">{{borderTile.label}}</span></span>
          </span>
        </div>
        <div class="game-map__dame-pool">
        <span v-for="(remainingDame, remainingDamesIndex) in remainingDames"
              class="tile"
              :id="'dame-pool-tile-'+remainingDamesIndex"
              @dragover="allowDrop($event)"
              @dragenter="dragEnter($event)"
              @dragleave="dragLeave($event)"
              @drop="drop($event, rowIndex = undefined, tileIndex = undefined, remainingDamesIndex = remainingDamesIndex)">
          <img src="/static/dame_transparent.png"
               draggable="true"
               @dragstart="drag($event)"
               :id="'dame_'+(remainingDamesIndex + 1)"
               :data-dame-number="(remainingDamesIndex + 1)"
               :class="[(!remainingDame.occupiedByDame >= 0) ? 'tile--occupied' : '', (remainingDame.threatened) ? 'tile--threatened' : '']">
        </span>
        </div>

      </div>
    </div>
  </div>
</template>

<script>
  import axios from 'axios'

  export default {
    name: 'dames-problem',
    data() {
      return {
        /* create map of background board tiles, that indicate the position of the set dames.
        *  an static method was chosen for approachability to the code */
        borderTiles: [[{label: ''},{label: 'A'},{label: 'B'},{label: 'C'},{label: 'D'},{label: 'E'},{label: 'F'},{label: 'G'},{label: 'H'},{label: '' }],
                      [{label: '8'},{/*not rendered*/},{},{},{},{},{},{},{},                                                               {label: '8'}],
                      [{label: '7'},{},{},{},{},{},{},{},{},                                                                               {label: '7'}],
                      [{label: '6'},{},{},{},{},{},{},{},{},                                                                               {label: '6'}],
                      [{label: '5'},{},{},{},{},{},{},{},{},                                                                               {label: '5'}],
                      [{label: '4'},{},{},{},{},{},{},{},{},                                                                               {label: '4'}],
                      [{label: '3'},{},{},{},{},{},{},{},{},                                                                               {label: '3'}],
                      [{label: '2'},{},{},{},{},{},{},{},{},                                                                               {label: '2'}],
                      [{label: '1'},{},{},{},{},{},{},{},{},                                                                               {label: '1'}],
                      [{label: ''},{label: 'A'},{label: 'B'},{label: 'C'},{label: 'D'},{label: 'E'},{label: 'F'},{label: 'G'},{label: 'H'},{label: ''}]],
        tiles: [[{},{},{},{},{},{},{},{}],
                [{},{},{},{},{},{},{},{}],
                [{},{},{},{},{},{},{},{}],
                [{},{},{},{},{},{},{},{}],
                [{},{},{},{},{},{},{},{}],
                [{},{},{},{},{},{},{},{}],
                [{},{},{},{},{},{},{},{}],
                [{},{},{},{},{},{},{},{}]],
        remainingDames: [],
        title: '8-Dames-Problem'
      }
    },
    mounted() {
      var self = this;
      this.init();
    },
    methods: {
      init () {
        this.setupTiles();
        this.setupDames();
      },
      setupTiles() {
        this.tiles = this.tiles.map((row, ri) => {
          return row.map((tile, index) => {
            if (index === 3 && (ri === 4 || ri === 1)) return { occupiedByDame: (ri === 1) ? 2 : 1, threatened: false }
            return { occupiedByDame: -1, threatened: false }
          })
        })
      },
      setupDames() {
        this.remainingDames = [{}, {}, {}, {}, {}, {}, {}, {}].map((dame, dameIndex) => {
            return { occupiedByDame: (dameIndex+1) }
        })
      },
//      handle drag and dropping the dames manually
      allowDrop(evt) {
//        console.log('allowDrop evt: ', evt, evt.target);
        evt.target.tagName === "SPAN" && !evt.target.classList.contains('tile--threatened') && evt.preventDefault();
      },
      dragLeave(evt) {
        evt.target.parentNode.removeAttribute('drop-active');
        evt.dataTransfer.setData("text", evt.target.id);
      },
      dragEnter(evt) {
//        console.log('allowDrop evt: ', evt.target);
        evt.target.parentNode.setAttribute('drop-active', true);
      },
      drag(evt) {
        console.log('drag evt: ', evt, evt.srcElement)
        evt.dataTransfer.setData("text", evt.target.id)
      },
      drop(evt, rowIndex, tileIndex, remainingDamesIndex) {
//        console.log('drop evt: ', evt, evt.target)

        evt.target.removeAttribute('drop-active')
        evt.target.parentNode.removeAttribute('drop-active')
        evt.preventDefault() //actually do drop the dame img on the tile

        var data = evt.dataTransfer.getData("text")
        var draggedElement = document.getElementById(data)
        var dameNumber = (draggedElement.dataset['dameNumber']) ? parseInt(draggedElement.dataset['dameNumber']) : -1;
        var targetTile = null
        if (rowIndex !== undefined && tileIndex !== undefined) {
          targetTile = this.tiles[rowIndex][tileIndex]
          targetTile.occupiedByDame = dameNumber
          console.log('this.tiles: ', this.tiles)
        }


        console.log('this.remainingDames: ', this.remainingDames);
        if (remainingDamesIndex !== undefined) {
          console.log('this.remainingDames: ', this.remainingDames, remainingDamesIndex)
          this.remainingDames[remainingDamesIndex].occupiedByDame = parseInt(dameNumber)
        }


//        its a dame pool tile, so set remove its
//        evt.srcElement.parentNode.id.indexOf('dame-pool-tile') > -1
        console.log('draggedElement: ', evt, targetTile, dameNumber ,draggedElement);
        evt.target.appendChild(draggedElement);
      }
    }
  }
</script>

<style lang="stylus">
  .problem
    position relative
    margin auto auto
    width 80vh
    heigth 100%
    padding-top 0
    z-index 1
  .game-map
    position relative
    margin: 10vh 8vh
    &__field
      .tile
        z-index auto
    &__dame-pool
      position absolute
      top auto
      bottom -9vh
      left 8vh
      /*top 8vh*/
      .tile
        /*clear both*/
        float right
        img:hover
          cursor grab
    &__border
      position absolute
      top -8vh
      left -8vh
      z-index -1
      & .game-map__dame-pool
        .tile
          background transparent
      .tile
        position relative
        background #1f1f1f
        float left
        color white
        text-align center
        &-label
          top 50%
          transform translate(-50%, -50%)
          position absolute
          left 50%
        &--background
          opacity 0
      .row
        width 80vh
        max-width 80vh
        max-height 8vh
  .row
    /*border 1px stroke red*/
    width 64vh
    max-width 64vh
    max-height 8vh
  .tile
    height 8vh
    width 8vh
    display inline-block
    &--white
      background #ededed
    &--black
      background #2f2f2f

    &--background
      background blue
      float left
    &:hover
      .tile--occupied
        background darkorange
        cursor grab
      .tile--threatened
        /*cursor pointer*/
        background darkred
        cursor no-drop
    &--occupied
      width 8vh
      height 8vh
    &--threatened
      width 8vh
      height 8vh
      /*background yellow*/
      background darkred
    &[drop-active=true]
      background darkred


/*box-shadow: 0px 1px 35px 100px blue;*/
</style>

