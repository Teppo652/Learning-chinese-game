<template>
  <div>
    <section class="section">
        <div class="container has-text-centered"> 
            
            <h5 id="top" class="subtitle is-4">Learn chinese 
              <button @click.prevent="showSettingsPanel = showSettingsPanel ? false : true, 
              !showSettingsPanel ? getWordSet() : ''" class="button iconButton">
                <span v-show="!showSettingsPanel" class="icon is-big is-primary">
                  <i class="fas fa-sliders-h"></i>
                </span>
                <span v-show="showSettingsPanel" class="icon is-big is-primary">
                  <i class="fas fa-check"></i>
                </span> 
              </button>  
            </h5>

            <!-- ========================= game settings section ========================= -->
            <div class="columns is-centered">
                <div class="column is-4 is-4-desktop">
                    <span v-if="showSettingsPanel">
                        <!-- hskLevels - not in use yet - only HSK1 set (150 words) is available
                        <label class="label">HSP level</label>
                        <div class="field has-addons">
                            <p v-for="l in hskLevels" 
                                    :key="l"
                                    :value="l"
                                    @click="hskLevel = $event.target.value" 
                                    class="control"
                                >
                                <a class="button is-warning">
                                    {{ l }}
                                </a>
                            </p>
                        </div>-->

                        <!-- :key="index" :value="m" -->
                        <label class="label">Number of words</label>
                        <div class="field has-addons">
                            <p v-for="c in cardSetSizes" 
                                    :key="c"
                                    @click="cardSetSize = c"             
                                    class="control"
                                >
                                <a class="button is-warning">
                                    {{ c }}
                                </a>
                            </p> {{ cardSetSize }}
                        </div>

                        <!-- displayTypes -->
                        <label class="label">Display</label>
                        <div class="field has-addons">
                            <p v-for="(d, index) in displayTypes" 
                                    :key="d"
                                    @click="displayType = index" 
                                    class="control"
                                >
                                <a class="button is-warning">
                                    {{ d }}
                                </a>
                            </p> {{ displayType }}
                        </div>
                    </span>
                </div>
            </div>

            <!-- ========================= game section - card grid ========================= -->
            <span style="float:left">{{ percentage}}%</span> &nbsp; 
            Correct:  {{ correct }} &nbsp; 
            Wrong: {{ wrong }}<br><br>
            <h5 id="meaning" class="subtitle is-3">{{ meaning }}</h5>
            <div class="columns is-centered">
                <div id="gameBoard" class="column is-5 is-5-desktop"
                    v-show="!showSettingsPanel">  
                    <div v-for="(a, index) in set" :key="index" 
                    class="column box tag is-warning"
                    :class="[
                        a.display == 'char' ? 'char' : 'text', 
                        (a.display.substr(0,1) + a.id) == selection1 || (a.display.substr(0,1) + a.id) == selection2 ? 'selected' : '',
                        hide.find(s => (s == (a.display.substr(0,1))+a.id)) ? 'hide': '',
                        corrects.find(s => (s == (a.display.substr(0,1))+a.id)) ? 'correct': (wrongs.find(s => (s == (a.display.substr(0,1))+a.id)) ? 'wrong': ''),
                        a.color == 'red' ? 'charColorRed' : (a.color == 'blue' ? 'charColorBlue' : '')
                        ]"
                    @click="hide.find(s => (s == (a.display.substr(0,1))+a.id)) 
                        ? null 
                        : wordClicked(a.display.substr(0,1) + a.id)" 
                    >{{ a.content }}
                    </div>
                </div> 
            </div>

        </div>
    </section>
   </div>
</template>

<script>
import hsk1 from './hsk1.json'
export default {    
  name: 'game',
  data () {
    return {
        showSettingsPanel: false,
        hskLevels: [1,2,3,4,5,6],
        hskLevel: 1,
        displayTypes: ["char-pinyin","char-eng","eng-pinyin"],
        displayType: 0,
        repeatsPerGroup: 20,
        cardSetSizes: [2,4,6,9,12,18,24],
        cardSetSize: 6,
        numberOfWordsFromGroup1: [3,6,10,12],
        numberOfWordsFromGroup2: [2,4,5,8], 
        numberOfWordsFromGroup3: [1,2,3,4],
        set: null,
        setIds: [],
        selections: [],
        selection1: '',
        selection2: '',
        hide: [],
        answer: null,
        numAnswered: 0,
        correct: 0,
        wrong: 0,        
        corrects: [],
        wrongs: [],
        percentage: 0,
        meaning: null,
        color: null,
        words: hsk1
    }
  },
  methods: {
      wordClicked(id) {   
          this.meaning = null;               
          if(!this.selection1) {
              this.selection1 = id;
          } else { 
              // check if logically clicked wrong - two pinyins or two characters - do nothing              
              if( this.selection1.substr(0,1) == id.substr(0,1)) { return; }
              this.numAnswered++;
              this.selection2 = id; // needed for styling
              // check if cards match
              let comp1 = this.selection1.substr(1);
              let comp2 = this.selection2.substr(1);              
              if(comp1 == comp2) {
                  // correct answer given                  
                  // show meaning text on top of screen
                  switch(this.displayType) {
                    case 0: this.meaning = this.words.find(s => (s.id == comp1)).eng; break;
                    case 1: this.meaning = this.words.find(s => (s.id == comp1)).pinyin; break;
                    case 2: this.meaning = this.words.find(s => (s.id == comp1)).char; break;
                  }                  
                  this.correct++;
                  this.answer = 'correct';
                  this.corrects.push(this.selection1, this.selection2);                  
                  if(comp1 == comp2) { this.hide.push(this.selection1, this.selection2); }

                  // check if screen is empty - load next set
                  if(this.hide) {
                    if(parseInt(this.cardSetSize)*2 == this.hide.length) {                         
                        // this.answer = 'correct - ALL DONE';         
                        // wait x sec
                        let self = this;
                        setTimeout(function () {
                            // clean up
                            self.inEnglish = null;
                            self.selection1 = '';
                            self.selection2 = '';
                            self.selections = [];
                            self.answer = null;
                            self.hide = [];
                            // load and show next set (screen)
                            self.set = [];
                            self.getWordSet();
                        }, 2000);                        
                    } 
                  }
              } else {
                  // wrong answer
                  this.wrongs.push(this.selection1, this.selection2);
                  this.wrong++;
                  this.answer = 'wrong';
              }
              this.calculatePercentage();

            // wait x secs and clear selections
            let self = this;
            setTimeout(function () {
                //alert('next');
                self.selection1 = '';
                self.selection2 = '';
                self.selections = [];
                self.corrects = [];
                self.wrongs = [];
            }, 750);
          }
      },
      calculatePercentage() {
          // percentage of correct answers
        this.percentage = (this.correct * 100 / this.numAnswered).toFixed(0);        
      },
      shuffle() {
          let arr = this.set;
          for (let i = 0; i < 100; i++) {
            arr.sort(() => Math.random() - 0.5);
          }
          this.set = arr;
      },
      sortIntoGroups(a, b) {
            // Use toUpperCase() to ignore character casing
            const setA = a.display.toUpperCase();
            const setB = b.display.toUpperCase();
            let comparison = 0;
            if (setA > setB) {
                comparison = 1;
            } else if (setA < setB) {
                comparison = -1;
            }
            return comparison;
      },
      getWordSet() {
          const words = this.words;
          const setSize = this.cardSetSize;
          let set = [];
          let setIds = [];
          var displayA = this.displayTypes[this.displayType].split("-")[0];
          var displayB = this.displayTypes[this.displayType].split("-")[1];
          while (parseInt(set.length)/2 < setSize) {
            let rndNum = Math.floor(Math.random() * parseInt(words.length)) + 1;  // returns a random integer from 1 to 10
            let colorA = '';
            let colorB = '';
            if(setIds.indexOf(words[rndNum].id) == -1) {
            
            // card
            let contentA = words[rndNum].displayA;
            switch(displayA) {
                case "char":   contentA = words[rndNum].char; colorA = 'red'; break;
                case "pinyin": contentA = words[rndNum].pinyin; colorA = 'blue';  break;
                case "eng":    contentA = words[rndNum].eng; break;
            }
            // pair to card
            let contentB = words[rndNum].displayB;
            switch(displayB) {
                case "char":   contentB = words[rndNum].char; colorB = 'red'; break;
                case "pinyin": contentB = words[rndNum].pinyin; colorB = 'blue';  break;
                case "eng":    contentB = words[rndNum].eng; break;
            }
            set.push({
                "id": words[rndNum].id,
                "content": contentA,
                "display": displayA,
                "color": colorA
                });
            set.push({
                "id": words[rndNum].id,
                "content": contentB,
                "display": displayB,
                "color": colorB
                });

                setIds.push(words[rndNum].id);
            } 
            this.set = set;            
            this.shuffle(); // suffle cards
            this.set.sort( this.sortIntoGroups); 
            this.setIds = setIds;
          }           
      }
  },
  created() {
      this.getWordSet();
  }
}
</script>

<style scoped>
#meaning {
    font-family: 'Noto Sans SC', sans-serif;
    height: 50px;
    text-transform: capitalize;
}

svg.fa-clock {
    margin: 0 2px 1px 14px;
}
.iconButton {
  background-color: #efefef !important;
  float: right;
  padding: 0 3px;
}
.iconButton span {
  margin:0 5px !important;
  padding: 0;
}
.iconButton svg path {
   /* fill: #ffe681; */
   fill: #f5f5f5;
 }



.tags .tag {
    font-size: 2rem;
    font-weight: 600;
}
.radio-buttons-as-buttons input {
  display: none;
}
.tags .tag {
    font-size: unset;
}
.tag {
    height: 100px;
    margin: 1px;
    width: 32%;
    background-color: #cdcdcd !important;
    border-radius: 0 !important;
}
.tag.selected {
    color: #fff;
    background-color: #383838 !important;
}
.tag:hover:not(.hide) {
    background-color: #d6d6d6 !important;
}
.char {
    font-family: 'Noto Sans SC', sans-serif;
    font-size: 2.2rem;
    float: left;
}
.text {
    font-size: 1.2rem;
    float: left;
}
#gameBoard div:hover {
    /* cursor: pointer; */
    cursor: url('..\assets\cursor.png') 4 12, auto;
}
.tag.selected.wrong {
    background-color: rgb(223, 107, 107) !important;
    color: #fff !important;
}
.tag.selected.correct {
    background-color: green !important;
    color: #fff !important;
}
.charColorRed {
    color: red !important;
} 
.charColorBlue {
    color: blue !important;
}
.hide {
    background-color: #fff !important;
    font-size: 0 !important;
}

/* Styles for Vertical screen */
@media all and (orientation:portrait) {
    .section {
        padding: .5rem .2rem;
    }
    .columns {
        margin-left: -.2rem;
        margin-right: -.7rem;
        margin-top: -.75rem;
    }
    .column {
        padding: 0;
    }
    .box {
        width:32%;
    }
}
/* Styles for Horizontal screen */
@media all and (orientation:landscape) {
    #top { display: none; }
    .section {
        padding: .5rem .2rem;
    }
    .columns {
        margin-left: -.15rem;
        margin-right: -.75rem;
        margin-top: -.75rem;
    }
    .column {
        padding: .2rem;
    }
    .box {
        width: 24%;
    }
}


</style>