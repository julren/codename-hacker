<script setup lang="ts">

import {ref} from "vue";


const output = ref<string[]>([])
const isLogginIn = ref(false)
const hasCorrectUser = ref(false)
const hasCorrectPw = ref(false)
const showImg = ref(false)
const isInFolder = ref(false)
const location = ref('')
const prompt = ref('')
const promptInput = ref()
const terminalContainer = ref<HTMLElement>()

function onEnter() {
  showImg.value = false;
  const promptValue = promptInput.value.innerText.trim()
  const next = nextText(promptValue.toLowerCase())
  if (!promptValue) return;

  output.value.push(`> ${promptValue}`)
  output.value.push(next)
  promptInput.value.innerText = '';
  setPromptValue('')
  setTimeout(() => {
    promptInput.value?.scrollIntoView()
  },0)
}

function nextText(text: string): string {
  if (hasCorrectPw.value) {
    return `Du bist ein echter Hackerman!`
  }

  if (text === 'h' || text === 'help' ) {
    return help;
  }

  if (text === 'cd recipes' || text === 'cd /recipes') {
    isInFolder.value = true
    location.value = 'recipes'
    return ''
  }

  if (text === 'cd ..') {
    isInFolder.value = false
    location.value = ''
    return ''
  }

  if (text.includes('cd')) {
    return "usage 'cd /directory' or 'cd ..' to go back"
  }

  if (text === 'ls' && isInFolder.value) {
    return `snack.txt`
  }

  if (text === 'ls') {
    return `welcome.txt   /recipes`
  }

  if (text === 'cat welcome.txt') {
    return `
We need to go deeper. Our friend god normally uses this system.
To access his account we need a password.
He is known to be forgetful sometimes. Maybe we can figure it out?
Use the (l)ogin command to try to get in.`
  }

  if (text === 'cat snack.txt') {
    return `
My Favorite Snack

Ingredients
- Peanuts

Steps:
1. Make Peanuts
2. Done!

I hope I didn't forget anything?`
}


  if (text.includes('cat')) {
    return `usage 'cat <filename>'`
  }

  if (text === 'c' || text === 'cancel') {
    isLogginIn.value = false;
    hasCorrectUser.value = false;
    hasCorrectPw.value = false;
    return 'canceled';
  }

  if ((text === 'l' || text === 'login') && !isLogginIn.value) {
    isLogginIn.value = true;
    return `user:`
  }


  if (isLogginIn.value && !hasCorrectUser.value) {
    if (text === 'god') {
      hasCorrectUser.value = true;
      return `password:`
    }
    return `user does not exist. try again or exit with (c)ancel.
user:
`
  }


  if (isLogginIn.value && hasCorrectUser.value) {
    if (text === 'chocolate') {
      isLogginIn.value = false
      hasCorrectPw.value = true;
      showImg.value = true;
      setTimeout(() => promptInput.value?.scrollIntoView(false), 5)
      return `🎉 You're in! Du bist ein echter Hackerman!`

    }
    return `wrong password. try again or exit with (c)ancel`
  }

  return `unrecognized command ${text}. Type (h)elp for help`
}

const help =
`ls    list files in current directory
cat   print file content, usage 'cat filename.txt'
cd    change directory, usage 'cd /directory' or 'cd ..' to go back
login login to a user account`

function setPromptValue(value: string) {
  promptInput.value.innerText = value;
}

</script>

<template>
  <main id="container" @click="promptInput.focus()">
    <div id="terminal" ref="terminalContainer">
    <pre>
      <output>
╭━━━┳━━━┳━━━┳━━━┳━╮╱╭┳━━━┳━╮╭━┳━━━╮
┃╭━╮┃╭━╮┣╮╭╮┃╭━━┫┃╰╮┃┃╭━╮┃┃╰╯┃┃╭━━╯
┃┃╱╰┫┃╱┃┃┃┃┃┃╰━━┫╭╮╰╯┃┃╱┃┃╭╮╭╮┃╰━━╮
┃┃╱╭┫┃╱┃┃┃┃┃┃╭━━┫┃╰╮┃┃╰━╯┃┃┃┃┃┃╭━━╯
┃╰━╯┃╰━╯┣╯╰╯┃╰━━┫┃╱┃┃┃╭━╮┃┃┃┃┃┃╰━━╮
╰━━━┻━━━┻━━━┻━━━┻╯╱╰━┻╯╱╰┻╯╰╯╰┻━━━╯
╭╮╱╭┳━━━┳━━━┳╮╭━┳━━━┳━━━╮
┃┃╱┃┃╭━╮┃╭━╮┃┃┃╭┫╭━━┫╭━╮┃
┃╰━╯┃┃╱┃┃┃╱╰┫╰╯╯┃╰━━┫╰━╯┃
┃╭━╮┃╰━╯┃┃╱╭┫╭╮┃┃╭━━┫╭╮╭╯
┃┃╱┃┃╭━╮┃╰━╯┃┃┃╰┫╰━━┫┃┃╰╮
╰╯╱╰┻╯╱╰┻━━━┻╯╰━┻━━━┻╯╰━╯

It's hacking time. Press (h)elp to find out about about available commands.
<span class="line" v-for="outputText of output">{{outputText}}</span></output></pre>
      <Transition>
      <img src="https://i.ytimg.com/vi/KEkrWRHCDQU/maxresdefault.jpg" v-if="showImg">
        </Transition>


    </div>
<div class="prompt">
  <div>>{{location}}</div>
  <div style="display: flex;">
    <div class="promptInput" ref="promptInput"  @keydown.enter.prevent @keyup.enter.prevent="onEnter" autofocus contenteditable="true"></div>
    <div class="caret"></div>
  </div>
<!--  <input ref="promptInput" v-model="prompt" @keyup.enter="onEnter" autofocus/>-->
</div>
  </main>
</template>

<style>

#terminal {
  overflow-y: scroll;
}

#terminal::-webkit-scrollbar {
  display: none;
}
  #container {
    padding-top: 2.5rem;
    min-height: 100%;
}

  .prompt {
    padding: 1rem 0;
  }

  input {
    caret-color: transparent;
  }
 .line {
   line-height: 1.2rem;
  display: block;
 }
 img {
   margin-top: 2rem;
   width: 100%;
 }

.v-enter-active,
.v-leave-active {
  transition: opacity 2s ease;
}

.v-enter-from,
.v-leave-to {
  opacity: 0;
}


.promptInput {
  background-color: transparent;
  border: transparent;
  caret-color: transparent;
  color: white;
  font: 1rem Inconsolata, monospace;
  text-shadow: 0 0 5px #C8C8C8;
  scroll-margin: 4rem;
}
.promptInput:focus {
  background: transparent;
  border: transparent;
  outline: none;
}

.caret {
  content: ' ';
  width: 1ch;
  height: 100%;
background-color: white;
  animation: blink 1200ms linear infinite;

}

@keyframes blink {
  0% {
    background: #ffffff;
  }
  49% {
    background: #ffffff;
  }
  60% {
    background: transparent;
  }
  99% {
    background: transparent;
  }  100% {
       background: #ffffff;
     }
}
</style>
