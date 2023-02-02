<template>
  <div class="bg-dark text-center" style="height: 100%">
    <h1 class="text-light p-3">AI IMAGE GENERATOR</h1>

    <div class="input container">
      <div class="box d-flex justify-content-end">
        <svg
          @click="speechRecogniser"
          xmlns="http://www.w3.org/2000/svg"
          width="20"
          height="20"
          id="mic"
          fill="red"
          class="bi bi-mic-fill mt-2 p"
          viewBox="0 0 16 16"
          style="position: absolute"
        >
          <path d="M5 3a3 3 0 0 1 6 0v5a3 3 0 0 1-6 0V3z" />
          <path
            d="M3.5 6.5A.5.5 0 0 1 4 7v1a4 4 0 0 0 8 0V7a.5.5 0 0 1 1 0v1a5 5 0 0 1-4.5 4.975V15h3a.5.5 0 0 1 0 1h-7a.5.5 0 0 1 0-1h3v-2.025A5 5 0 0 1 3 8V7a.5.5 0 0 1 .5-.5z"
          />
        </svg>
        <input
          type="text"
          name=""
          id="text-input"
          v-model="inputText"
          class="form-control"
          placeholder="Enter Something To generate AI based Images"
        />
      </div>
      <button
        type="submit"
        class="btn btn-danger w-100"
        @click.prevent="imageGeneratorHandler(inputText)"
      >
        Generate Image
      </button>
      <input
        type="text"
        name=""
        id="text-input"
        v-model="inputText2"
        autocomplete=""
        class="form-control"
        placeholder="Enter Something To generate AI based code or text"
      />
      <button
        type="submit"
        class="btn btn-danger w-100"
        @click.prevent="textGeneratorHandler(inputText2)"
      >
        Generate Text
      </button>
    </div>
    <div v-if="flag" class="spinner-border text-danger" role="status">
      <span class="visually-hidden">Loading...</span>
    </div>
    <div class="container" v-else-if="imgUrl">
      <img :src="imgUrl" alt="no image Found" class="img-fluid" />
    </div>
    <div class="erorr" v-else>nothing Found</div>
  </div>
  <!-- <pre
    class="card card-body mt-3 rounded-3 mx-3 bg-dark text-light"
    v-if="generatedText1"
  >
  <button @click="copyCode">copy</button>


  <code class="text-break language-javascript" id="code-block"> 
    {{ generatedText1 }}
  </code>
</pre> -->

  <ssh-pre
    dark
    reactive
    copy-button
    language="js"
    v-for="item in codeArray"
    :key="item.index"
  >
    <template #copy-button>
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="25"
        height="25"
        fill="white"
        class="bi bi-clipboard"
        viewBox="0 0 16 16"
      >
        <path
          d="M4 1.5H3a2 2 0 0 0-2 2V14a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V3.5a2 2 0 0 0-2-2h-1v1h1a1 1 0 0 1 1 1V14a1 1 0 0 1-1 1H3a1 1 0 0 1-1-1V3.5a1 1 0 0 1 1-1h1v-1z"
        />
        <path
          d="M9.5 1a.5.5 0 0 1 .5.5v1a.5.5 0 0 1-.5.5h-3a.5.5 0 0 1-.5-.5v-1a.5.5 0 0 1 .5-.5h3zm-3-1A1.5 1.5 0 0 0 5 1.5v1A1.5 1.5 0 0 0 6.5 4h3A1.5 1.5 0 0 0 11 2.5v-1A1.5 1.5 0 0 0 9.5 0h-3z"
        />
      </svg>
    </template>
    {{ item.text }}
  </ssh-pre>
  <p
    v-if="generatedText2"
    class="card card-body text-center mt-3 rounded-3 mx-3 bg-dark text-light"
  >
    {{ generatedText2 }}
  </p>
</template>
<script>
import { Configuration, OpenAIApi } from "openai";
import SshPre from "simple-syntax-highlighter";
import "simple-syntax-highlighter/dist/sshpre.css";

export default {
  name: "App",
  data() {
    return {
      inputText: "",
      copiedText: "",
      imgUrl: "",
      openai: null,
      flag: false,
      inputText2: "",
      generatedText1: "",
      generatedText2: "",
      codeArray: [],
    };
  },
  components: { SshPre },
  created() {
    const configuration = new Configuration({
      apiKey: import.meta.env.VITE_API_KEY,
    });
    this.openai = new OpenAIApi(configuration);
  },

  methods: {
    speechRecogniser: async function () {
      const btn = document.querySelector("#mic");

      window.SpeechRecognition =
        window.webkitSpeechRecognition || window.SpeechRecognition;
      const recognition = new SpeechRecognition();

      recognition.onstart = () => {
        btn.innerText = "Listening...";
      };

      recognition.onresult = (event) => {
        const current = event.resultIndex;
        this.inputText2 = event.results[current][0].transcript;
        btn.innerHTML =
          '<svg xmlns="http://www.w3.org/2000/svg"  width="16" height="16"  fill="red"  class="bi bi-mic-fill"  viewBox="0 0 16 16"  > <path d="M5 3a3 3 0 0 1 6 0v5a3 3 0 0 1-6 0V3z" /> <path   d="M3.5 6.5A.5.5 0 0 1 4 7v1a4 4 0 0 0 8 0V7a.5.5 0 0 1 1 0v1a5 5 0 0 1-4.5 4.975V15h3a.5.5 0 0 1 0 1h-7a.5.5 0 0 1 0-1h3v-2.025A5 5 0 0 1 3 8V7a.5.5 0 0 1 .5-.5z"  /> </svg>';
      };

      recognition.start();
    },
    imageGeneratorHandler: async function (val) {
      this.flag = true;
      if (val) {
        const response = await this.openai.createImage({
          prompt: this.inputText,
          n: 1,
          size: "1024x1024",
        });
        console.log(response.data.data[0].url);
        this.imgUrl = response.data.data[0].url;
        this.flag = false;
      }
    },
    checkForCodeRequest(inpValue) {
      const keywords = [
        "generate code",
        "write code",
        "write a code",
        "code for",
        "how to",
        "route",
        "router",
        "java",
        "create code",
        "code to",
        "make code",
        "program for",
        "develop code",
        "script for",
        "build code",
        "produce code",
        "coding for",
        "create program",
        "scripting for",
        "code generation",
        "generate programming",
        "write a program",
        "coding help",
        "programming assistance",
      ];

      const pattern = new RegExp(`\\b(${keywords.join("|")})\\b`, "i");
      return pattern.test(inpValue);
    },
    textGeneratorHandler: async function (val) {
      const bool = this.checkForCodeRequest(val);
      console.log("boolean", bool);
      if (bool) {
        this.generatedText2 = "";
        const completion = await this.openai.createCompletion({
          model: "text-davinci-003",
          prompt: this.inputText2,
          max_tokens: 1024,
          n: 10,
          temperature: 1.0,
        });
        this.codeArray = completion.data.choices;
        console.log(completion.data.choices);
        this.generatedText1 = completion.data.choices[0].text;
      } else {
        this.generatedText1 = "";
        const completion = await this.openai.createCompletion({
          model: "text-davinci-003",
          prompt: this.inputText2,
          max_tokens: 1024,
          n: 5,
          temperature: 1.0,
        });

        this.generatedText2 = completion.data.choices[0].text;
      }
    },
    copyCode: function () {
      const textarea = document.createElement("t");
      var range = document.createRange();
      range.selectNode(document.getElementById("code-block"));
      window.getSelection().removeAllRanges(); // clear current selection
      window.getSelection().addRange(range); // to select text
      document.execCommand("copy");
      window.getSelection().removeAllRanges();
      alert("copied");
      // to deselect

      // const codeBlock = document.querySelector("#code-block");

      // create a hidden textarea element
      // textarea.value = codeBlock.textContent;
      // document.body.appendChild(textarea);
      // codeBlock.select();
      // document.execCommand("copy");
      // textarea.remove();
    },
  },
};
</script>
<style scoped>
code {
  font-family: "Source Code Pro", monospace;
  line-height: 18px;
}
</style>
