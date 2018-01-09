<template>
  <div class="problem">
    <h1>{{ title }}</h1>
    <!--v-show="border.label" -->
    <div class="game-map">

      <div class="game-map__field">
        <div v-for="(row, rowIndex) in tiles" :class="(rowIndex%2 === 0) ? 'row row--even' :'row row--odd'">
        <span v-for="(tile, tileIndex) in row"
              class="tile"
              :class="[(tileIndex%2 === 0) ? 'tile--even' : 'tile--odd',
                      ((tileIndex%2 === 0 && rowIndex%2 === 0) || (tileIndex%2 === 1 && rowIndex%2 === 1)) ? 'tile--white' : 'tile--black']"
        >
          <img v-if="tile.occupied"
               src="/static/dame_transparent.png"
               :class="[(tile.occupied) ? 'tile--occupied' : '', (tile.threatened) ? 'tile--threatened' : '']">

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
        <span v-for="(remainingDame, remainingDamesIndex) in remainingDames" class="tile">
          <img src="/static/dame_transparent.png"
               :class="[(!remainingDame.placed) ? 'tile--occupied' : '', (remainingDame.threatened) ? 'tile--threatened' : '']">
        </span>
        </div>

      </div>
    </div>
  </div>
</template>

<script>
  import axios from 'axios'

  export default {
    name: 'hello',
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
              if (index === 3 && (ri === 4 || ri === 1)) return { occupied: true, threatened: false }
              return { occupied: false, threatened: false }
            })
        })
        console.log('this.tiles: ', this.tiles);
      },
      setupDames() {
        this.remainingDames = [{}, {}, {}, {}, {}, {}, {}, {}].map((el) => {
            return { placed: false }
        })
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
      /*top 8vh*/
      .tile
        /*clear both*/
        float right
        img:hover
          cursor pointer
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
        cursor pointer
        background green
    &--occupied
      width 8vh
      height 8vh
    &--threatened
      width 8vh
      height 8vh
      /*background yellow*/
      background darkred


/*box-shadow: 0px 1px 35px 100px blue;*/
</style>

