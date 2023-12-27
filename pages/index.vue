<template>
    <div class="container">
        <h1>Music Translator</h1>

        <div>Source Code: <a href="https://github.com/luantber" target="_blank"> Github repo</a> </div>

        <p>Este traductor incrementa en el número deseado de semitonos las notas insertadas. </p>
        <p>Las notas se insertan en formato (do,re, do# , reb ) . Acepta mayusculas o minusculas, con la posibilidad de añadir un asterisco (*) o varios (**) para subir una octava o guiones (-) para bajar una octava.</p>
        <p>Usa espacios y saltos de línea para separar tus notas</p>
        <p>Por ejemplo:</p>
        <pre>
La la la la sib DO* 
FA* MI* SOL* DO* DO*
Sib la sib DO* RE*
DO*# DO*# RE*# FA* RE*# DO*# DO**
Sib la-
        </pre>


        <div class="panel">
            <div>
                <div>Original</div>
                <textarea class="panel__box" v-model="original_text"></textarea>
            </div>
            <div>
                <div>Resultado</div>
                <textarea class="panel__box" v-model="translated_text"></textarea>
            </div>
        </div>
        <div class="interface">
            <div>Desplazamiento(incremento) en Semitonos:</div>
            <div>
                <input class="interface__semitonos" type="number" min="-10" max="10" v-model="semitones_tosum">
                <button class="interface__button" @click="translate">Traducir</button>
            </div>
    </div>
    </div>
</template>

<script setup lang="ts">
const semitones_tosum = ref(3)
const original_text = ref('')
const translated_text = ref('')

const equivalent_notes : Record<string,number> = {
    "do": 1,
    "do#": 2,
    "reb": 2,
    "re": 3,
    "re#": 4,
    "mib": 4,
    "mi": 5,
    "fa": 6,
    "fa#": 7,
    "solb": 7,
    "sol": 8,
    "sol#": 9,
    "lab": 9,
    "la": 10,
    "la#": 11,
    "sib": 11,
    "si": 12
}

const equivalent_notes_reverse: Record<number,string>= {
    1: "do",
    2: "do#",
    3: "re",
    4: "re#",
    5: "mi",
    6: "fa",
    7: "fa#",
    8: "sol",
    9: "sol#",
    10: "la",
    11: "sib",
    12: "si"

}


function transpose(note: number, octave: number){
    let new_note = note + semitones_tosum.value
    let new_octave = octave
    if (new_note > 12){
        new_note = new_note - 12
        new_octave = octave + 1
    }
    if (new_note < 1){
        new_note = new_note + 12
        new_octave = octave - 1
    }
    return {
        note: new_note,
        octave: new_octave
    }

}


function translate_word(word: string){

    if (word == '') return ''

    let octave_up = word.split('*').length - 1
    let octave_down = word.split('-').length - 1

    

    if ((octave_up > 0) && (octave_down > 0)) {
        throw new Error("Invalid note")
    }

    let octave_number = octave_up - octave_down

    let clean_note = word.replace(/\*|-/g, '')
    clean_note = clean_note.toLowerCase()
    
    
    let number_note = equivalent_notes[clean_note]
    
    let new_note = transpose(number_note, octave_number)
    let new_note_name = equivalent_notes_reverse[new_note.note]

    let suffix = new_note.octave > 0 ? '*'.repeat(new_note.octave) : '-'.repeat(-new_note.octave)
    let new_word = new_note_name + suffix

    return new_word
}

function translate_line(line: string){
    let words = line.split(' ')
    let translated_words = words.map(translate_word)
    return translated_words.join(' ')
}

function translate(){
    // Split the text into lines
    let lines = original_text.value.split('\n')
    let translated_lines = lines.map(translate_line)

    translated_text.value = translated_lines.join('\n')

}

</script>

<style lang="sass">
@import "~/assets/sass/main.sass"


.container
    margin-left: 10%
    margin-right: 10%

.panel
    display: grid
    grid-template-columns: 1fr 1fr
    margin-bottom: 20px

    &__box
        width: 100%
        height: 200px
        resize: none
        border: 1px solid black
        padding: 10px
        box-sizing: border-box

.interface
    display: flex
    flex-direction: column
    align-items: center

    &__semitonos
        width: 50px
        margin-right: 10px
        font-size: 15px
      
    &__button
        width: 100px
        height: 30px
        border: none
        border-radius: 5px
        background-color: $color-primary
        cursor: pointer
        color: white
</style>