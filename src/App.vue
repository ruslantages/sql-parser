<template>
  <main>
    <div class="sql-parser-wrapper">
      <div class="sql-parser-wrapper__side input-side">
        <textarea
          v-model="textareaInputValue"
          ref="textArea"
          class="sql-input"
        />
        <button @click="onSubmit">
          Parse
        </button>
      </div>
      <div class="sql-parser-wrapper__side output-side">
        <div class="sql_output" v-html="output"/>
      </div>
    </div>

  </main>
</template>

<script setup lang="ts">
import {ref} from "vue";
import uniqolor from 'uniqolor';

const sqlPinkCommands = ["begin", "delete", "from", "where", "insert", "into", "values", "update", "commit", "set", "select"]
const sqlOrangeCommands = ["in"]

const sqlCommandsAfterWhichCarryOnKeys = ["from", "where", "update", "set", "select", "into"]

const textArea = ref<HTMLTextAreaElement | null>(null);
const textareaInputValue = ref<string>("INSERT INTO \"units_shipments\" (\"product_guid\", \"oks_guid\", \"production_division_guid\", \"characteristics_guid\", \"measure_unit_type\", \"measure_unit_guid\", \"loaded_at\", \"deleted\") VALUES\n" +
  "('{\"name\": \"Ruslan\",\"age\": 26}', '5bd8e210-45c7-48fd-9581-e5f57ff9bc93', '362c10ea-4bf2-47bd-bb10-9d50357a9d50', NULL, 'Shipment', 'd9d12e98-c853-4474-aa5a-6ae508e7de30', to_timestamp((0)), '0'),\n" +
  "('f52907c7-2f1f-4686-9614-42d161c0da6d', '5bd8e210-45c7-48fd-9581-e5f57ff9bc93', '362c10ea-4bf2-47bd-bb10-9d50357a9d50', NULL, 'Spec', 'fe7da683-c82e-4d15-b645-842b95083932', to_timestamp((0)), '0'),\n" +
  "\n" +
  "('f52907c7-2f1f-4686-9614-42d161c0da6d', '3205ee35-7c8d-49fa-83ef-ebe1ce74b4a5', '5f0865a6-6b9d-42bf-a887-b46a14ceb9b8', NULL, 'Shipment', '3b7666e0-5d61-45e4-b162-28a74ad530d3', to_timestamp((0)), '0'),\n" +
  "('f52907c7-2f1f-4686-9614-42d161c0da6d', '3205ee35-7c8d-49fa-83ef-ebe1ce74b4a5', '5f0865a6-6b9d-42bf-a887-b46a14ceb9b8', NULL, 'Spec', '051c09db-c7a7-4b19-a8b9-2c77f295d708', to_timestamp((0)), '0');");
const output = ref<string>("");

function putSqlCommand(word: string) {
  if (sqlPinkCommands.includes(word.toLowerCase())) {
    // сама команда
    output.value += `<span style="color: pink">${word}</span> `
  } else if (sqlOrangeCommands.includes(word.toLowerCase())) {
    // сама команда
    output.value += `<span style="color: orange">${word}</span> `
  } else {
    output.value += word + " "
  }
}

function isItKeyFromCommand(wordsFromRowArr: Array<string>, wordIndex: number) {
  return Boolean(wordsFromRowArr[wordIndex - 1]) && sqlCommandsAfterWhichCarryOnKeys.includes(wordsFromRowArr[wordIndex - 1].toLowerCase())
}

function putKeyFromCommand(word: string) {
  output.value += `<span style="color: blue">${word}</span> `
}

function onSubmit() {
  output.value = ""

  const rows = textareaInputValue.value.split(";");
  rows.filter(Boolean).forEach((row, commandIdx) => {
    row += ";"
    row = row.replace(/(\s|\n)+/g, " ")
    let insertKeys: Array<string> = [];
    let insertValues: Array<Array<string>> = [];

    if (row.toLowerCase().includes("insert") && row.toLowerCase().includes("values")) {
      row = row.replace(/\)\s?values\s?\(/i, ") VALUES (").replace(/,(?!\s("|')?)/g, ", ");

      const insertKeysSubstr = row.match(/\(.*\)(?=\s?values\s?\()/ig)
      if (insertKeysSubstr && insertKeysSubstr[0]) {
        insertKeys = insertKeysSubstr[0]
          .replace(/^\(|\)$/g, "")
          .split(", ")
          .map(el => el.replace(/^("|')|("|')$/g, ""))
      }

      const insertValuesSubstr = row.match(/\sVALUES\s.*(?=;)/)
      if (insertValuesSubstr && insertValuesSubstr[0]) {
        insertValues = insertValuesSubstr[0]
          .replace(/\s?VALUES\s?/, "")
          .replace(/\),\s?\(/g, "),__(")
          .split(",__")
          .map(el => el
            .replace(/^\(|\)$/g, "")
            .replace(/,\s(?!.*(\{|\}))/g, ",__")
            .split(",__")
            .map(nestedEl => nestedEl
              .replace(/^("|')|("|')$/g, "")
            )
          )
      }

      if ((!insertKeys.length || !insertValues.length) || (insertKeys.length && insertValues.length && !insertValues.every(el => el.length === insertKeys.length))) {
        alert("Не удалось распарсить скрипт");
        return;
      }

      const randomColors = insertKeys.map((el, idx) => {
        return uniqolor(new Date().getTime() + el + idx)
      })

      const regExArrayBeforeKeys = row.match(/insert\sinto\s?("|')?\w*("|')?\s?/i)
      let substrBeforeKeys = ""
      if (regExArrayBeforeKeys && regExArrayBeforeKeys[0]) {
        substrBeforeKeys = regExArrayBeforeKeys[0]
      }

      output.value += substrBeforeKeys.trim() + "</br></br>("

      insertKeys.forEach((el, idx) => {
        output.value += `<span
          data-key-idx="${idx}"
          data-row-idx="${commandIdx}"
          style="
            background: ${randomColors[idx].color};
            color: ${randomColors[idx].isLight ? "black" : "white"};
          "
        >${el}</span>`
        if (idx !== insertKeys.length - 1) {
          output.value += ", "
        }
      })

      output.value += ")</br></br>VALUES"
      insertValues.forEach((el, idx) => {
        output.value += "</br></br>("

        el.forEach((keysValues, keysValuesIdx) => {
          output.value += `<span
          data-key-idx="${keysValuesIdx}"
          data-row-idx="${commandIdx}"
          style="
            background: ${randomColors[keysValuesIdx].color};
            color: ${randomColors[keysValuesIdx].isLight ? "black" : "white"};
          "
        >${keysValues}</span>`
          if (keysValuesIdx !== el.length - 1) {
            output.value += ", "
          }
        })

        if (idx === insertValues.length - 1) {
          output.value += ");"
        } else {
          output.value += "),"
        }
      })
    } else {
      const wordsFromRow = row.trim().split(" ").map(el => el.trim());

      wordsFromRow.filter(el => el !== "").forEach((word, wordIndex, wordsFromRowArr) => {
        if (
          word.startsWith('"')
          || word.startsWith("'")
          || word.startsWith("(")
          || word.endsWith(",")
          || word.endsWith(")")
        ) {
          // какое-то значение ключа или название ключа
          if (isItKeyFromCommand(wordsFromRowArr, wordIndex)) {
            putKeyFromCommand(word)
          } else {
            output.value += word + " "
          }
        } else {
          // какое-то слово из команды
          if (isItKeyFromCommand(wordsFromRowArr, wordIndex)) {
            // название ключа или название таблицы из команды
            putKeyFromCommand(word)
          } else {
            putSqlCommand(word)
          }
        }
      })
    }
    output.value += "</br></br></br>"
  })
}
</script>

<style>
main {
  width: 100%;
  max-width: 95%;
  margin-left: auto;
  margin-right: auto;
}

.sql-parser-wrapper {
  display: flex;
  align-items: flex-start;
}

.sql-parser-wrapper__side {
  width: 50%;
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  padding: 20px;
  box-sizing: border-box;
}

.sql-parser-wrapper__side textarea,
.sql-parser-wrapper__side .sql_output {
  width: 100%;
  height: 400px;
  border: 2px solid blue;
  border-radius: 8px;
  font-size: 18px;
  line-height: 22px;
  font-family: monospace;
  padding: 8px;
  box-sizing: border-box;
}

.input-side textarea {
  width: calc(100% - 150px);
  resize: vertical;
}

.input-side button {
  width: 125px;
  height: 40px;
  border: 2px solid blue;
  background: none;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  font-size: 20px;
  text-transform: uppercase;
}

.input-side button:hover {
  background: rgba(0, 0, 255, 0.20);
}

.output-side .sql_output {
  border-color: green;
  resize: both;
  overflow: auto;
  line-height: 1.5;
}
</style>
