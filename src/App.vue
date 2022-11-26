<template>
  <main>
    <div class="container">
      <form class="user-input" @submit.prevent="encode">
        <h2>Raw input</h2>
        <textarea v-model="rawInput" @input="inputError = ''"></textarea>
        <div class="input-error">
          <p v-if="inputError">{{ inputError }}</p>
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
      rawInput: "That's one small step for man, one giant leap for mankind.",
      encodedString: "",
      matrix: [] as string[][],
      paddedMatrix: [] as string[],
      inputError: "",
    };
  },
  mounted() {
    this.encode();
  },
  methods: {
    encode() {
      const normalized = this.rawInput.replace(/[^a-zA-Z]/g, "").toLowerCase();

      if (normalized.length > 64) {
        this.inputError =
          "Normalized text should be 64 characters long at most";
        return;
      }

      const size = Math.round(Math.sqrt(normalized.length));
      this.matrix = this.spreadIntoMatrix(size, normalized);
      this.encodedString = this.encodeFromMatrix(size, this.matrix);
      this.paddedMatrix = this.generatePaddedMatrix(size, this.encodedString);
    },

    spreadIntoMatrix(size: number, normalized: string): string[][] {
      let cursor = 0;
      const matrix = [];
      for (let row = 0; row < size; row++) {
        matrix.push(new Array(size));
        for (let col = 0; col < size; col++) {
          matrix[row][col] = normalized[cursor] || "";
          cursor++;
        }
      }
      return matrix;
    },

    encodeFromMatrix(size: number, matrix: string[][]): string {
      let encodedStr = "";
      for (let col = 0; col < size; col++) {
        for (let row = 0; row < size; row++) {
          encodedStr += matrix[row][col] || "";
        }
      }
      return encodedStr;
    },

    generatePaddedMatrix(size: number, str: string): string[] {
      const trailingSpaces = str.length % size;
      const chunks = [];
      for (let row = 0; row < size; row++) {
        let chunk = "";
        let start = row * size;
        let amount = size;
        if (trailingSpaces && row >= trailingSpaces) {
          amount--;
          if (row >= trailingSpaces) {
            start = row * size - row + trailingSpaces;
          }
        }
        for (let i = start; i < start + amount; i++) {
          chunk += str[i] || "";
        }
        if (row >= trailingSpaces) chunk += " ";
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

button {
  display: block;
  padding: 0.5rem 1rem;
  min-width: 160px;
}

.encoded-string {
  margin-bottom: 1rem;
}

.padded-matrix {
  background: #333;
  color: white;
  padding: 1rem;
  font-family: "Courier New", Courier, monospace;
  margin-bottom: 1rem;
}

.row-col-error,
.input-error {
  min-height: 28px;
  font-size: 14px;
  color: darkred;
  font-weight: bold;
}
</style>
