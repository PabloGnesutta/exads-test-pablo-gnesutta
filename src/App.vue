<template>
  <main>
    <div class="container">
      <form class="user-input" @submit.prevent="encode">
        <h2>Raw input</h2>
        <textarea v-model="rawInput" @input="inputError = ''"></textarea>
        <div class="input-error">
          <p v-if="inputError">{{ inputError }}</p>
        </div>
        <div class="dimension-select">
          <div>
            <label>Rows</label>
            <input v-model="rows" type="number" placeholder="rows" />
          </div>
          <div>
            <label>Cols</label>
            <input v-model="cols" type="number" placeholder="cols" />
          </div>
        </div>
        <div class="row-col-error">
          <p v-if="!rowColRatioIsValid">Invalid row to column ratio</p>
        </div>
        <button type="submit">ENCODE</button>
      </form>
      <div v-if="encodedString" class="result">
        <div class="encoded-string">
          <h3>Encoded string:</h3>
          {{ encodedString }}
        </div>
        <div>
          <h3>Padded matrix:</h3>
          <div class="padded-matrix">
            <div v-for="item in paddedMatrix" :key="item">{{ item }}</div>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>

<script lang="ts">
export default {
  data() {
    return {
      // rawInput: "una amiga esta yendo a un barco.",
      rawInput: "That's one small step for man, one giant leap for mankind.",
      encodedString: "",
      paddedMatrix: [] as string[],
      inputError: "",
      rows: 7,
      cols: 7,
    };
  },
  mounted() {
    this.encode();
  },
  computed: {
    rowColRatioIsValid() {
      return this.cols >= this.rows && this.cols - this.rows <= 1;
    },
  },
  methods: {
    encode() {
      if (!this.rowColRatioIsValid) return;
      const lower = this.rawInput.toLowerCase();
      const normalized = lower.replace(/[^a-zA-Z]/g, "");
      if (normalized.length > 64) {
        this.inputError =
          "Normalized text should be 64 characters long at most";
        return;
      }
      const matrix = this.spreadIntoMatrix(this.rows, this.cols, normalized);
      this.encodedString = this.encodeFromMatrix(this.rows, this.cols, matrix);
      this.paddedMatrix = this.generatePaddedMatrix(
        this.rows,
        this.cols,
        this.encodedString
      );
    },

    spreadIntoMatrix(
      rows: number,
      cols: number,
      normalized: string
    ): string[][] {
      let cursor = 0;
      const matrix = [];
      for (let row = 0; row < rows; row++) {
        matrix.push(new Array(cols));
        for (let col = 0; col < cols; col++) {
          matrix[row][col] = normalized[cursor];
          cursor++;
        }
      }

      return matrix;
    },

    encodeFromMatrix(rows: number, cols: number, matrix: string[][]): string {
      let encodedStr = "";
      for (let col = 0; col < cols; col++) {
        for (let row = 0; row < rows; row++) {
          encodedStr += matrix[row][col] || "";
        }
      }
      return encodedStr;
    },

    generatePaddedMatrix(rows: number, cols: number, str: string): string[] {
      const trailingSpaces = str.length % cols;
      console.log(str);
      const chunks = [];
      for (let col = 0; col < cols; col++) {
        let chunk = "";
        let start = col * cols;
        let amount = cols;
        if (trailingSpaces && col >= trailingSpaces) {
          amount--;
          if (col > trailingSpaces) {
            start = col * cols - col + trailingSpaces;
          }
        }
        console.log(start, amount);
        for (let i = start; i < start + amount; i++) {
          chunk += str[i] || "";
        }
        if (col >= trailingSpaces) chunk += " ";
        chunks.push(chunk);
      }
      return chunks;
    },
  },
};
</script>

<style scoped>
main {
  display: flex;
  align-items: center;
  justify-content: center;
}

.container {
  max-width: 1200px;
}

.user-input {
  margin-bottom: 2rem;
}
textarea {
  display: block;
  resize: vertical;
  width: 100%;
  min-height: 80px;
}

.dimension-select {
  display: flex;
  gap: 1rem;
  margin: 0.5rem 0;
}

label {
  display: block;
  font-size: 14px;
  font-weight: bold;
}

button {
  display: block;
  padding: 0.5rem 1rem;
  min-width: 160px;
  /* margin-left: auto; */
}

.encoded-string {
  margin-bottom: 1rem;
}

.padded-matrix {
  background: #333;
  color: white;
  padding: 1rem;
  font-family: "Courier New", Courier, monospace;
}

.row-col-error,
.input-error {
  min-height: 28px;
  font-size: 14px;
  color: darkred;
  font-weight: bold;
}
</style>
