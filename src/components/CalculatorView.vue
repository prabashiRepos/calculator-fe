<template>
  <div id="app">
    <div class="calculator">
      <input id="output" type="text" :class="nameClass"
             v-model="output" v-on:change="nameChange" readonly/>

      <div class="buttons">
        <button class="clear"
                @click="clearOutput">AC
        </button>
        <button class="clear"
                @click="getOperation('+/-')">+/-
        </button>
        <button class="clear"
                @click="getOperation('%')">%
        </button>
        <button class="operator"
                @click="getOperation('/')">
          <fa icon="fa-solid fa-divide"/>
        </button>

        <button class="number"
                @click="getNumber('7')">7
        </button>
        <button class="number"
                @click="getNumber('8')">8
        </button>
        <button class="number"
                @click="getNumber('9')">9
        </button>
        <button class="operator"
                @click="getOperation('*')">
          <fa icon="fa-solid fa-multiply"/>
        </button>

        <button class="number"
                @click="getNumber('4')">4
        </button>
        <button class="number"
                @click="getNumber('5')">5
        </button>
        <button class="number"
                @click="getNumber('6')">6
        </button>
        <button class="operator"
                @click="getOperation('-')">
          <fa icon="fa-solid fa-minus"/>
        </button>

        <button class="number"
                @click="getNumber('1')">1
        </button>
        <button class="number"
                @click="getNumber('2')">2
        </button>
        <button class="number"
                @click="getNumber('3')">3
        </button>
        <button class="operator"
                @click="getOperation('+')">
          <fa icon="fa-solid fa-plus"/>
        </button>

        <button class="zero"
                @click="getNumber('0')">0
        </button>
        <button class="number"
                @click="getNumber('.')">.
        </button>
        <button class="operator"
                @click="updateOutput()">
          <fa icon="fa-solid fa-equals"/>
        </button>

      </div>
    </div>
  </div>
</template>

<script>
import {ref} from 'vue';
import axios from 'axios';

const output = ref(0);
const previousOutput = ref(0);
const nameClass = ref('output');

export default {
  setup() {
    const nameChange = () => {
      if (output.value.length === 7) {
        nameClass.value = 'output-s';
      } else if (output.value.length > 7) {
        nameClass.value = 'output-ex-s';
      } else {
        nameClass.value = 'output';
      }
    }

    const getNumber = (number) => {
      nameChange();

      output.value == 0 && number == 0 ? (output.value = 0) :
          output.value === 0 && number !== 0 ? (output.value = number) : (output.value += number);
    }

    const getOperation = (operationSymbol) => {
      nameChange();
      if (operationSymbol === '+/-' && output.value === 0) {
        output.value = 0;
      } else if (operationSymbol === '+/-' && Math.sign(output.value) === 1) {
        output.value = -output.value;
      } else {
        output.value += operationSymbol;
      }
    }

    const updateOutput = () => {
      previousOutput.value = output.value;
      const removeDotRegex = /^[.]/ig;

      if (output.value.toString() !== '0') {
        output.value = (output.value).replace(removeDotRegex.exec(output.value), '0.');
        output.value = eval(String((output.value).replace('%', '/100')));

        nameChange();
        saveOutput();
      }
    }

    const clearOutput = () => {
      output.value = 0;
      nameChange();
    }

    const saveOutput = () => {
      axios
          .post('http://127.0.0.1:8000/api/calculator', {
            operation: previousOutput.value,
            result: output.value,
          })
          .then(async response => {
            const data = await response.data
            console.log(data);
            console.log(data.message);
            if (!data.success) {
              const error = (data && data.message) || response.status;
              console.log(data.message);
              return Promise.reject(error);
            }
          })
          .catch(error => {
            console.error("There was an error, ", error);
          });
    };

    return {
      output,
      getNumber,
      getOperation,
      updateOutput,
      clearOutput,
      nameChange,
      nameClass
    };
  },
};
</script>

<style scoped>
@import '../assets/css/calculator.css';
</style>