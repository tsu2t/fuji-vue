<script setup lang="ts">
import { ref,reactive } from 'vue'

const new_filename = defineModel("new_filename");
const files = reactive(load_files());
const filename = ref("");
const word = defineModel("word");
const meaning = defineModel("meaning");
const words = reactive(load_words());

function createfile(){
    let filename:string = String(new_filename.value);
    if(new_filename.value!=null && filename.length>0){
        create_new_file(filename);
        new_filename.value = null;
    }else{
        alert('input new file name');
    }
}
function create_new_file(name:string){
    let key:string = String(Date.now());
    files.set(key,name);
    localStorage.setItem('filelist' , JSON.stringify(Array.from(files.entries())));
}

function delete_file(key:string){
    let del_file:string = String(files.get(key));
    if(files.delete(key)){
        localStorage.setItem('filelist' , JSON.stringify(Array.from(files.entries())));
        localStorage.removeItem(del_file);
    }else{
        alert('delete error : ' + key);
    }
}

function load_files():Map<string,string>{
    // get filelist obj from storage
    let resultMap:Map<string,string>;
    let obj = localStorage.getItem('filelist');
    if(obj!=null){
        resultMap = new Map(JSON.parse(obj));
    }else{
        resultMap = new Map<string,string>();
    }
    return resultMap;
}

function load_words():Map<string,string>{
    // get filelist obj from storage
    let resultMap:Map<string,string>;
    let obj = localStorage.getItem(filename.value);
    if(obj!=null){
        resultMap = new Map(JSON.parse(obj));
    }else{
        resultMap = new Map<string,string>();
    }
    return resultMap;
}

function add_word(){
  if(word.value!=null&&meaning.value!=null){
    words.set(String(word.value) , String(meaning.value));
    localStorage.setItem(filename.value , JSON.stringify(Array.from(words.entries())));
    word.value = null;
    meaning.value = null;
  }else{
    alert('register error')
  }
}

function delete_word(delete_word:string){
  if(words.delete(delete_word)){
    localStorage.setItem(filename.value , JSON.stringify(Array.from(words.entries())));
  }else{
    alert('delete error2');
    return;
  }
}

function select_file(file_key:string){
    filename.value = String(files.get(file_key));
    const list = load_words();
    const keys = Object.keys(list);
    words.clear();
    // 詰め替え
    list.forEach((value,key)=>{
        words.set(key,value);
    });
}

'use strict'
function pronounce(p_word:string) {
    let u = new SpeechSynthesisUtterance();
    u.lang = 'en-US';
    u.text = p_word;
    speechSynthesis.speak(u);
}

</script>

<template>
    <h1 class="bg-gray-100">EitanGo!</h1>
    <div class="my-5">
        <h2>File List</h2>
        <ul>
            <li v-for="file in files" class="rounded-md bg-blue-100 p-4 m-1">
                <label @click="select_file(file[0])" class="w-2/3 float-left">{{ file[1] }}</label>
                <button @click="delete_file(file[0])" class="rounded-md bg-slate-800 py-1 px-1 border border-transparent text-center text-sm text-white transition-all shadow-md hover:shadow-lg focus:bg-slate-700 focus:shadow-none active:bg-slate-700 hover:bg-slate-700 active:shadow-none disabled:pointer-events-none disabled:opacity-50 disabled:shadow-none ml-2" type="button">Delete</button>
            </li>
            <li>
                <label class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">create new file</label>
                <input type="text" v-model="new_filename" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 w-1/2 p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500" placeholder="input new file name" required />
                <button @click="createfile" class="rounded-md bg-red-800 py-1 px-1 border border-transparent text-center text-sm text-white transition-all shadow-md hover:shadow-lg focus:bg-red-700 focus:shadow-none active:bg-red-700 hover:bg-red-700 active:shadow-none disabled:pointer-events-none disabled:opacity-50 disabled:shadow-none ml-2" type="button">Create</button>
            </li>
        </ul>
    </div>
    <div class="my-5">
        <h2>Word List</h2>
        <ul v-if="filename.length>0">
            <li  v-for="item in words" class="rounded-md bg-blue-100 p-4 m-1">
                <div @click="pronounce(item[0])" class="w-2/5 float-left">{{ item[0] }}</div>
                <div class="w-2/5 float-left">{{ item[1] }}</div>
                    <button @click="delete_word(item[0])" class="rounded-md bg-slate-800 py-1 px-1 border border-transparent text-center text-sm text-white transition-all shadow-md hover:shadow-lg focus:bg-slate-700 focus:shadow-none active:bg-slate-700 hover:bg-slate-700 active:shadow-none disabled:pointer-events-none disabled:opacity-50 disabled:shadow-none ml-2" type="button">Delete</button>
            </li>
            <li>
                <label for="email" class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">create new word</label>
                <input type="text" v-model="word" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 w-1/3 p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500" placeholder="input new word" required />
                <input type="text" v-model="meaning" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 w-1/3 p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500" placeholder="input meaning" required />
                <button @click="add_word" class="rounded-md bg-slate-800 py-1 px-1 border border-transparent text-center text-sm text-white transition-all shadow-md hover:shadow-lg focus:bg-slate-700 focus:shadow-none active:bg-slate-700 hover:bg-slate-700 active:shadow-none disabled:pointer-events-none disabled:opacity-50 disabled:shadow-none ml-2" type="button">Create</button>
            </li>
        </ul>
        <ul v-else>
            <li class="rounded-md bg-red-200 p-4 m-1">Choose File.</li>
        </ul>
    </div>
</template>
<!--
<script setup lang="ts">
import {ref,provide} from 'vue'
import FileList from './components/FileList.vue'
import WordList from './components/WordList.vue'

</script>

<template>
    <h1 class="bg-gray-100">EitanGo!</h1>
    <div class="my-5">
        <h2>File List</h2>
        <FileList />
    </div>
    <div class="my-5">
        <h2>Word List</h2>
        <WordList />
    </div>
</template>
-->