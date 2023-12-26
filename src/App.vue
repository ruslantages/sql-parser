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

const sqlPinkCommands = ["begin", "delete", "from", "where", "insert", "into", "values", "update", "commit", "set", "select"]
const sqlOrangeCommands = ["in"]

const sqlCommands = [...sqlPinkCommands, ...sqlOrangeCommands];
const findSqlCommandRegexp = new RegExp("(" + sqlCommands.join("|") + ")(?!\\w)", "ig")

const sqlCommandsEndsWithQuotes = [...sqlCommands.map(el => el + '"'), ...sqlCommands.map(el => el + "'")]
const sqlCommandsEndsWithOpenBracket = sqlCommands.map(el => el + "(")
const sqlCommandsEndsWithCarriageTransfer = sqlCommands.map(el => el + "\n")
const sqlCommandsStartsWithQuotes = [...sqlCommands.map(el => '"' + el), ...sqlCommands.map(el => "'" + el)]
const sqlCommandsStartsWithClosedBracket = sqlCommands.map(el => ")" + el)
const sqlCommandsStartsWithCarriageTransfer = sqlCommands.map(el => "\n" + el)

const sqlCommandWhichDoesntEndsWithSpace = [
  ...sqlCommandsEndsWithQuotes,
  ...sqlCommandsEndsWithOpenBracket,
  ...sqlCommandsEndsWithCarriageTransfer,
  ...sqlCommandsStartsWithQuotes,
  ...sqlCommandsStartsWithClosedBracket,
  ...sqlCommandsStartsWithCarriageTransfer,
]

const sqlCommandsAfterWhichCarryOnKeys = ["from", "where", "update", "set", "select", "into"]

const textArea = ref<HTMLTextAreaElement | null>(null);
const textareaInputValue = ref<string>("INSERT INTO \"units_shipments\" (\"product_guid\", \"oks_guid\", \"production_division_guid\", \"characteristics_guid\", \"measure_unit_type\", \"measure_unit_guid\", \"loaded_at\", \"deleted\") VALUES\n" +
  "('f52907c7-2f1f-4686-9614-42d161c0da6d', '5bd8e210-45c7-48fd-9581-e5f57ff9bc93', '362c10ea-4bf2-47bd-bb10-9d50357a9d50', NULL, 'Shipment', 'd9d12e98-c853-4474-aa5a-6ae508e7de30', to_timestamp((0)), '0'),\n" +
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

/*
let str = `INSERT INTO "values" ("product_guid", "oks_guid", "production_division_guid", "characteristics_guid", "measure_unit_type", "measure_unit_guid", "loaded_at", "deleted") values
('f52907c7-2f1f-4686-9614-42d161c0da6d', '5bd8e210-45c7-48fd-9581-e5f57ff9bc93', '362c10ea-4bf2-47bd-bb10-9d50357a9d50', NULL, 'Shipment', 'd9d12e98-c853-4474-aa5a-6ae508e7de30', to_timestamp((0)), '0'),
('f52907c7-2f1f-4686-9614-42d161c0da6d', '5bd8e210-45c7-48fd-9581-e5f57ff9bc93', '362c10ea-4bf2-47bd-bb10-9d50357a9d50', NULL, 'Spec', 'fe7da683-c82e-4d15-b645-842b95083932', to_timestamp((0)), '0'),

('f52907c7-2f1f-4686-9614-42d161c0da6d', '3205ee35-7c8d-49fa-83ef-ebe1ce74b4a5', '5f0865a6-6b9d-42bf-a887-b46a14ceb9b8', NULL, 'Shipment', '3b7666e0-5d61-45e4-b162-28a74ad530d3', to_timestamp((0)), '0'),
('f52907c7-2f1f-4686-9614-42d161c0da6d', '3205ee35-7c8d-49fa-83ef-ebe1ce74b4a5', '5f0865a6-6b9d-42bf-a887-b46a14ceb9b8', NULL, 'Spec', '051c09db-c7a7-4b19-a8b9-2c77f295d708', to_timestamp((0)), '0');`;

// Регулярное выражение для поиска данных в скобках после VALUES
let regex = /VALUES\s*([\s\S]*?)(?=;|(\), \())/g;

// Находим данные в скобках после VALUES с помощью регулярного выражения
let matches = str.replaceAll(" values ", " VALUES ").replaceAll(" values(", " VALUES (").replaceAll("values (", "VALUES (").replaceAll(")values", ") VALUES").replaceAll(") values", ") VALUES").replaceAll(")values(", ") VALUES (").match(regex);
console.log(matches.length)
if (matches) {
  console.log('Найденные элементы:');
  matches.forEach(match => {
    console.log(match.replaceAll("'", '"').replaceAll("VALUES (", "(").replaceAll(/,\n(?!\()/g, ", "));
  });
} else {
  console.log('Данные в скобках после VALUES не найдены');
}

* */

function onSubmit() {
  output.value = ""

  const rows = textareaInputValue.value.split(";");
  rows.filter(Boolean).forEach(row => {
    row += ";"
    row = row.replace(/(\s|\n)+/g, " ")
    let insertKeys: Array<string> = [];
    let insertValues: Array<Array<string>> = [];

    if (row.toLowerCase().includes("insert") && row.toLowerCase().includes("values")) {
      row = row.replace(/\)\s?values\s?\(/i, ") VALUES (").replace(/,(?!\s("|')?)/g, ", ");
      // eslint-disable-next-line no-console
      console.log("%c row2", "color: yellow", row)
      // .replace(/'/g, '"')
      const insertKeysSubstr = row.match(/\(.*\)(?=\s?values\s?\()/ig)
      if (insertKeysSubstr && insertKeysSubstr[0]) {
        insertKeys = insertKeysSubstr[0]
          .replace(/^\(|\)$/g, "")
          .split(", ")
          .map(el => el.replace(/^("|')|("|')$/g, ""))
      }
      // eslint-disable-next-line no-console
      console.log("%c insertKeys", "color: blue", insertKeys)

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
      // eslint-disable-next-line no-console
      console.log("%c insertValues", "color: green", insertValues);

      if ((!insertKeys.length || !insertValues.length) || (insertKeys.length && insertValues.length && !insertValues.every(el => el.length === insertKeys.length))) {
        alert("Не удалось распарсить скрипт");
        return;
      }
      // eslint-disable-next-line no-console
      console.log("%c row", "color: blue", row)
      const selectSubstringBeforeValuesRegex = /insert\sinto\s?\S+\s?\(.*\)(?=\s?values\s?\()/ig
      const substringBeforeValues = row.match(selectSubstringBeforeValuesRegex);
      if (substringBeforeValues && substringBeforeValues[0]) {
        row = row.replace(selectSubstringBeforeValuesRegex, "").trim();
      }
      // eslint-disable-next-line no-console
      console.log("%c row", "color: yellow", row)
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
    
  })
}
</script>

<style scoped>
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
}
</style>
