<script>
import GameBoard from '@/components/GameBoard.vue'

// eslint-disable-next-line vue/no-export-in-script-setup
export default {
  components: { GameBoard },
  data() {
    return {
      difficulty: 'beginner',
      customRows: 16,
      customColumns: 20,
      customMines: 30,
    }
  }
}
</script>

<template>
  <div>
    <header>
      <img alt="Minesweeper Flag" class="logo" src="./assets/minesweeper-flag.png" width="125" height="125" />

      <div class="wrapper">
        <h1>Welcome to Minesweeper</h1>
      </div>
    </header>
    <div class="setup">
      <fieldset>
        <legend>Difficulty</legend>
        <div>
          <input type="radio" name="beginner" id="beginner" value="beginner" v-model="difficulty">
          <label for="beginner">Beginner</label>
        </div>

        <div>
          <input type="radio" name="intermediate" id="intermediate" value="intermediate" v-model="difficulty">
          <label for="intermediate">Intermediate</label>
        </div>

        <div>
          <input type="radio" name="expert" id="expert" value="expert" v-model="difficulty">
          <label for="expert">Expert</label>
        </div>

        <div>
          <input type="radio" name="custom" id="custom" value="custom" v-model="difficulty">
          <label for="custom">Custom</label>

          <div class="custom-options" v-if="difficulty === 'custom'">
            <fieldset>
              <legend>Custom Options</legend>
              <div class="fields">
                <div class="labeled-field">
                  <label for="customRows">Rows</label>
                  <input
                    type="number"
                    name="customRows"
                    id="customRows"
                    min="10"
                    max="20"
                    v-model="customRows"
                  >
                </div>
                <div class="labeled-field">
                  <label for="customRows">Columns</label>
                  <input
                    type="number"
                    name="customColumns"
                    id="customColumns"
                    min="10"
                    max="30"
                    v-model="customColumns"
                  >
                </div>
                <div class="labeled-field">
                  <label for="customRows">Mines</label>
                  <input
                    type="number"
                    name="customMines"
                    id="customMines"
                    min="10"
                    max="150"
                    v-model="customMines"
                  >
                </div>
              </div>
            </fieldset>
          </div>
        </div>
      </fieldset>
    </div>
  </div>

  <main>
    <GameBoard :difficulty="difficulty" :custom-rows="customRows" :custom-columns="customColumns" :custom-mines="customMines" />
  </main>
</template>

<style lang="scss" scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
    padding-bottom: 20px;
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
.setup {
  margin: 20px;

  input[type=radio] {
    margin-right: 10px;
  }
}
.custom-options {
  .fields {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: center;
    width: 100%;

    .labeled-field {
      display: flex;
      flex-direction: column;
      flex: 1;
      align-items: center;
      justify-content: center;

      input {
        text-align: center;
        width: 80px;
        height: 40px;
        font-size: 1.2rem;
        border-radius: 8px;

        &::-webkit-inner-spin-button,
        &::-webkit-outer-spin-button {
          opacity: 1;
        }
      }
    }
  }
}
</style>
