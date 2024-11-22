<script setup>
import HelloWorld from './components/HelloWorld.vue'
import TheWelcome from './components/TheWelcome.vue'
</script>

<template>
  <Experiment title="Eksperiment për leximin" translate="no">
    <InstructionScreen :title="'Mirësevini në eksperimentin për gjurmimin e mausit'">
      <p>Në këtë studim ju do të lexoni pjesë të shkurtra teksti dhe do ti përgjigjeni pyetjeve të shkurtra rreth tyre. Megjithatë, teksti që do të shfaqet në ekran nuk do të jetë i dukshëm plotësisht. Në mënyrë që të shikoni tekstin duhet të lëvizni mausin mbi të. </p>
	  <p>Ju mund të shpenzoni aq kohë sa ju duhet për ta lexuar tekstin në mënyrë që ta kuptoni. Në momentin që keni përfunduar leximin, klikoni ‘Përfundo’ për të shfaqur pyetjen rreth tekstit. Pasi të ktheni përgjigjen e pyetjes klikoni ‘Vazhdo’ për të shfaqur tekstin e rradhës.</p>
	  <p>Kujdes: Për mbarëvajtjen e eksperimentit ju lutem tregoni vëmendje gjatë leximit të tekstit dhe përgjigjes së pyetjeve. </p>
	  </br>
	  <p>Faleminderit për pjesëmarrjen në eksperiment!</p>
	</InstructionScreen>

    <template v-for="(trial, i) of trials">
      <Screen :key="i" class="main_screen" :progress="i / trials.length">
        <Slide>
          <form>
            <input type="hidden" class="item_id" :value="trial.item_id">
            <input type="hidden" class="experiment_id" :value="trial.experiment_id">
            <input type="hidden" class="condition_id" :value="trial.condition_id">
          </form>
          <div class="oval-cursor"></div>
          <template>
            <div v-if="showFirstDiv" class="readingText" @mousemove="moveCursor" @mouseleave="changeBack">
              <template v-for="(word, index) of trial.text.split(' ')">
                <span :key="index" :data-index="index" >
                  {{ word }}
                </span>
              </template>
            </div>
            <div class="blurry-layer" style="opacity: 0.3; filter: blur(3.5px); transition: all 0.3s linear 0s;"> 
              {{trial.text}}
            </div>
          </template>
          <button v-if="showFirstDiv" style= "bottom:10%; transform: translate(-50%, -50%)" @click="toggleDivs" :disabled="!isCursorMoving">
          Përfundo
          </button>

          <div v-else style = "position:absolute; bottom:10%; text-align: center; width: 100%; min-width: -webkit-fill-available;" >
            <template>
              <form>
                <!-- comprehension questions and the response options -->
                <div>{{ trial.question.replace(/ ?["]+/g, '') }}</div>
                <template v-for='(word, index) of trial.response_options'>
                  <input :id="'opt_'+index" type="radio" :value="word" name="opt" v-model="$magpie.measurements.response"/>{{ word }}<br/>
                    <!-- <label :for="'opt_'+index"> {{ word }}&nbsp</label> -->
                </template>
              </form>
            </template>
          </div>
          
          <button v-if="$magpie.measurements.response" style="transform: translate(-50%, -50%)" @click="toggleDivs(); $magpie.saveAndNextScreen()">
            Vazhdo
          </button>
        </Slide>
      </Screen>
    </template>
<Screen>
  <p>1. Çfarë pajisje përdorët për të zhvilluar këtë eksperiment?</p>
    <MultipleChoiceInput
        :response.sync= "$magpie.measurements.device"
        orientation="horizontal"
        :options="['Computer Mouse', 'Computer Trackpad', 'Tjetër']" />
  <br>
  <br>
  <p>2. Cilën dorë përdorët gjatë eksperiemntit?</p>
    <MultipleChoiceInput
        :response.sync= "$magpie.measurements.hand"
        orientation="horizontal"
        :options="['Majtë', 'Djathtë', 'Të dyja']" />
  <button style= "bottom:30%; transform: translate(-50%, -50%)" @click="$magpie.saveAndNextScreen();">Përfundo</button>
</Screen>

    <SubmitResultsScreen />
  </Experiment>
</template>

<style scoped>
  .experiment {
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .main_screen {
    isolation: isolate;
    position: relative;
    width: 100%;
    height: auto;
    font-size: 18px;
    line-height: 40px;
  }
  .debugResults{
    width: 100%;
  }
  .readingText {
    /* z-index: 1; */
    position: absolute;
    color: white;
    text-align: left;
    font-weight: 450;
    cursor: pointer;
    padding-top: 2%;
    padding-bottom: 2%;
    padding-left: 11%;
    padding-right: 11%;
  }
  button {
    position: absolute;
    bottom: 0;
    left: 50%;
  }
  .oval-cursor {
    position: fixed;
    z-index: 2;
    width: 1px;
    height: 1px;
    transform: translate(-50%, -50%);
    background-color: white;
    mix-blend-mode: difference;
    border-radius: 50%;
    pointer-events: none;
    transition: width 0.5s, height 0.5s;
  } 
  .oval-cursor.grow.blank {
    width: 80px;
    height: 13px;
  }
  .oval-cursor.grow {
    width: 102px;
    height: 38px;
    border-radius: 50%;
    box-shadow: 30px 0 8px -4px rgba(255, 255, 255, 0.1), -30px 0 8px -4px rgba(255, 255, 255, 0.1);
    background-color: rgba(255, 255, 255, 0.3);
    background-blend-mode: screen;
    pointer-events: none;
    transition: width 0.5s, height 0.5s;
    filter:blur(3px);
  }
  .oval-cursor.grow::before {
    content: "";
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 70%;
    height: 70%;
    background-color: white;
    mix-blend-mode: normal;
    border-radius: 50%;
}
  .blurry-layer {
    position: absolute;
    pointer-events: none;
    color: black;
    text-align: left;
    font-weight: 450;
    padding-top: 2%;
    padding-bottom: 2%;
    padding-left: 11%;
    padding-right: 11%;
  }

  * {
    user-select: none; /* Standard syntax */
    -webkit-user-select: none; /* Safari */
    -moz-user-select: none; /* Firefox */
    -ms-user-select: none; /* Internet Explorer/Edge */
    }
</style>
