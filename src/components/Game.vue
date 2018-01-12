<template>
  <div class="problem">
    <h1>{{ title }}</h1>

    <span class="game-settings">
      <!-- displays the button to find start the solution finder or manually search for solutions -->
      <v-btn class=""
             @click="findOneSolutions">find a solution</v-btn>
      <v-btn class=""
             @click="findAllSolutions">find all solutions</v-btn>
      <v-btn class=""
             @click="solveAlone">{{ (solve.mode === 'auto') ? 'solve alone' : 'reset' }}</v-btn>
    </span>

    <div class="game-map">

      <div class="game-map__field">
        <div v-for="(row, rowIndex) in tiles" :class="(rowIndex%2 === 0) ? 'row row--even' :'row row--odd'">
        <span v-for="(tile, tileIndex) in row" :key="rowIndex+'_'+tileIndex"
              class="tile"
              :id="'tile_'+rowIndex+'_'+tileIndex"
              :class="[(tileIndex%2 === 0) ? 'tile--even' : 'tile--odd',
                      ((tileIndex%2 === 0 && rowIndex%2 === 0) || (tileIndex%2 === 1 && rowIndex%2 === 1)) ? 'tile--white' : 'tile--black']"
              @dragover="allowDrop($event)"
              @dragenter="dragEnter($event)"
              @dragleave="dragLeave($event)"
              @drop="drop($event, rowIndex, tileIndex)"
        >
          <!--<img v-if="tile.occupiedByDame >= 0"-->
               <!--draggable="true"-->
               <!--@dragstart="dragStart($event)"-->
               <!--src="/static/dame_transparent.png"-->
               <!--:id="'dame_' + tile.occupiedByDame"-->
               <!--:data-dame-number="tile.occupiedByDame"-->
               <!--:class="[(tile.occupiedByDame >= 0) ? 'tile&#45;&#45;occupied' : '', (tile.threatened) ? 'tile&#45;&#45;threatened' : '']">-->

        </span>
        </div>
      </div>
      <div class="game-map__border">
        <div v-for="(borderRow, borderRowIndex) in borderTiles" :key="borderRowIndex"
             class="row">
          <span v-for="(borderTile, borderTileIndex) in borderRow" :key="borderRowIndex + '_'+ borderTileIndex"
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
               @dragstart="dragStart($event)"
               :id="'dame_'+(remainingDamesIndex + 1)"
               :data-dame-number="(remainingDamesIndex + 1)"
               :class="[(remainingDame.occupiedByDame >= 0) ? 'tile--occupied' : '', (remainingDame.threatened) ? 'tile--threatened' : '']">
        </span>
        </div>

      </div>
    </div>
  </div>
</template>

<script>
import Vue from 'vue'
import axios from 'axios'
import dragAndDropMixin from './DragAndDrop.vue'
var range = (a, b) => {while(a--)b[a]=a+1;return b}
//  ES6 style [...Array(N+1).keys()].slice(1)

export default {
  name: 'dames-problem',
  mixins: [dragAndDropMixin],
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
      title: '8-Dames-Problem',
      solve: { mode: 'auto', message: 'solve yourself'},
      N: 8, /*determines the complexity of the problem*/
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
      /* remove all rendered dames*/
      var $dames = document.querySelectorAll('img[data-dame-number]');
      [].forEach.call($dames, (dame) => {
        var dameSelector = '#dame-pool-tile-'+(parseInt(dame.dataset['dameNumber'])-1);
        var $damePoolTile = document.querySelector(dameSelector);
        $damePoolTile.appendChild(dame);
      })

      /* place new dames on the game field */
      this.remainingDames = [{}, {}, {}, {}, {}, {}, {}, {}].map((dame, dameIndex) => {
        return { occupiedByDame: (dameIndex+1) }
      })
      this.$forceUpdate();
    },
    solveAlone() {
      if(this.solve.mode === 'auto') {
        this.solve.mode = 'alone'
        this.init()
      }else {
        this.init()
        console.log('this.tiles:  ', this.tiles[7][7].occupiedByDame);
      }
    },
    findOneSolutions() {
      var self = this;
      this.solve.mode = 'auto'
      this.init();

      /* create N x N Array[][]*/
      let field = [...Array(self.N).keys()].map((row) => {
          /* put zeros in it to mark the field empty */
          return [...Array(self.N).keys()].map(() => { return 0; })
      })
      console.log('field: ', field);
      /* search for a solution algorithmically */
      if ( this.findSolution(field, 0) === false ){
        console.warn("no solutions were found");
        return ;
      }
      var solution = field
      this.prettyPrintNDamesSolution(solution)
      /* append the solution to the game-field */

    },
    findAllSolutions() {
      this.solve.mode = 'auto'
      this.init();
    },
    prettyPrintNDamesSolution(solution){

      var alphabetArray = [...Array(this.N).keys()].map((charIndex) => {
        return String.fromCharCode(charIndex + 65);
      })
      console.log('At least one solution found: \r\n\r\n  ', alphabetArray.join(',')+ '\r\n' +
                  solution.map((row, ind)=>{ return ind+' [' + row.toString() + ']'; }).join('\r\n'));
    },
    /*The main algorithm to find a solution algorithmically
     * 1.) iterate over each row and simulate placing dame by marking the tile with an 1 if the position is safe,
     *     otherwise stop this iteration
     * 2.) if the current[row][col] position is safe an doesnt attack other dames on the western part of the chess field,
     *     we increment the currently watched column rekursively and search for a solution in the next column
     * 3.) when that search doesnt succeed, remove the previously placed dame by setting a 0 on the dame [row][col] field
     * 4.) as soon as all rekursive paths return and all rows have been inspected,
     *     a solution is either found, or no solution exists
     *
     * @returns Solution to N-Dames-Problem */
    findSolution(field, currentCol) {
      var self = this;
      if(currentCol >= self.N) return true;

      /* iterate over each row in column currentCol */
      for (let currentRow = 0; currentRow < self.N; currentRow++){
//        console.log('try placind dame on   row[%s][%s]col ', currentRow, currentCol);
        if(self.positionIsSafe(field, currentRow, currentCol)){
          /*queen is currently safe at [currentRow][currentCol], so place a dame at this position by marking it with a 1*/
          field[currentRow][currentCol] = 1

          /* deep dive into the successive solutions to find out
           *  if the current position of the current dame is ok and part of a solution
           *  if yes, we return the sub-solution, otherwise we delete the dame from [currentRow][currentCol]
           *  and go on with the next rowIndex by returning false*/
          if(self.findSolution(field, currentCol+1)) return true;
          field[currentRow][currentCol] = 0
        }
      }
      /* no solution was found for all rows -> go back to currentCol-1*/
      var noSolution = JSON.parse(JSON.stringify(field));
//      console.log('no solution for currentCol: '/*, currentCol, ' ... no solution with: ', noSolution.map((row)=>{return '[' + row.toString() + ']\r\n'; })*/ );
      return false
    },
    positionIsSafe(field, currentRow, currentCol) {
      var self = this;
      let i, j;

      /* is there a dame set on this row? */
      for (i = 0; i < currentCol; i++)
        if (field[currentRow][i])
          return false;

      /* inspect north west diagonal path and cancel if dame with an 1 is found */
      for (i=currentRow, j=currentCol; i>=0 && j>=0; i--, j--)
        if (field[i][j])
          return false;

      /* inspect south west diagonal path and cancel if dame with an 1 is found */
      for (i=currentRow, j=currentCol; j>=0 && i<self.N; i++, j--)
        if (field[i][j])
          return false;

      /* no need to look to the right, because the right is always empty or the end of the chess board */
      return true;
    },
    marksTreathenedTiles() {


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
  .game-settings
    position relative
    width 100%
    height 100%
    padding auto auto
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

