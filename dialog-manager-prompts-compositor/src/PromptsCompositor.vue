<template>
  <div class="min-h-screen bg-gray-100 py-8 px-4 sm:px-6 lg:px-8">
    <div class="max-w-4xl mx-auto">
      <h1 class="text-3xl font-bold text-center text-gray-900 mb-8">Prompts Compositor</h1>
      
      <div class="bg-white shadow-md rounded-lg p-6 mb-6">
        <h2 class="text-xl font-semibold mb-4">Crear/Editar Prompt</h2>
        <input
          v-model="promptTitle"
          class="w-full p-2 border rounded-md mb-4"
          placeholder="Título del prompt"
        />
        <textarea
          v-model="promptText"
          @input="updatePreview"
          class="w-full h-32 p-2 border rounded-md mb-4"
          placeholder="Escribe tu prompt aquí. Usa {{variable}} para insertar variables."
        ></textarea>
        
        <div class="mb-4">
          <h3 class="text-lg font-medium mb-2">Variables</h3>
          <div v-for="(value, key) in variables" :key="key" class="flex items-center mb-2">
            <input
              :value="key"
              @input="updateVariableName(key, $event.target.value)"
              class="border rounded-md p-2 w-1/3 mr-2"
              placeholder="Nombre de variable"
            />
            <input
              v-model="variables[key]"
              @input="updatePreview"
              class="border rounded-md p-2 w-2/3"
              :placeholder="`Valor para ${key}`"
            />
          </div>
          <button
            @click="addVariable"
            class="bg-blue-500 text-white px-4 py-2 rounded-md hover:bg-blue-600 transition-colors"
          >
            Añadir Variable
          </button>
        </div>
        
        <div class="flex justify-between">
          <button
            @click="savePrompt"
            class="bg-green-500 text-white px-4 py-2 rounded-md hover:bg-green-600 transition-colors"
          >
            Guardar Prompt
          </button>
          <button
            @click="clearForm"
            class="bg-gray-500 text-white px-4 py-2 rounded-md hover:bg-gray-600 transition-colors"
          >
            Nuevo Prompt
          </button>
        </div>
      </div>
      
      <div class="bg-white shadow-md rounded-lg p-6 mb-6">
        <h2 class="text-xl font-semibold mb-4">Vista Previa del Prompt</h2>
        <div class="bg-gray-100 p-4 rounded-md whitespace-pre-wrap">{{ previewText }}</div>
      </div>
      
      <div class="bg-white shadow-md rounded-lg p-6">
        <h2 class="text-xl font-semibold mb-4">Prompts Guardados</h2>
        <ul class="divide-y divide-gray-200">
          <li v-for="prompt in savedPrompts" :key="prompt._id" class="py-4">
            <div class="flex items-center justify-between">
              <span class="text-lg font-medium">{{ prompt.title }}</span>
              <div>
                <button
                  @click="loadPrompt(prompt)"
                  class="text-blue-500 hover:text-blue-700 mr-2"
                >
                  Editar
                </button>
                <button
                  @click="deletePrompt(prompt._id)"
                  class="text-red-500 hover:text-red-700"
                >
                  Eliminar
                </button>
              </div>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue'
import axios from 'axios'

const API_BASE_URL = 'http://localhost:8000/api'  // Ajusta esto a la URL de tu API FastAPI

const promptTitle = ref('')
const promptText = ref('')
const variables = reactive({})
const savedPrompts = ref([])
const currentPromptId = ref(null)

const addVariable = () => {
  const newKey = `variable${Object.keys(variables).length + 1}`
  variables[newKey] = ''
}

const updateVariableName = (oldKey, newKey) => {
  if (newKey && newKey !== oldKey) {
    variables[newKey] = variables[oldKey]
    delete variables[oldKey]
    updatePreview()
  }
}

const previewText = computed(() => {
  let preview = promptText.value
  for (const [key, value] of Object.entries(variables)) {
    preview = preview.replace(new RegExp(`{{${key}}}`, 'g'), value || `{{${key}}}`)
  }
  return preview
})

const updatePreview = () => {
  // Esta función se llama automáticamente cuando cambia el texto del prompt o los valores de las variables
}

const savePrompt = async () => {
  try {
    const promptData = {
      title: promptTitle.value,
      promptText: promptText.value,
      variables: variables
    }
    
    if (currentPromptId.value) {
      await axios.put(`${API_BASE_URL}/prompts/${currentPromptId.value}`, promptData)
    } else {
      await axios.post(`${API_BASE_URL}/prompts`, promptData)
    }
    
    await loadSavedPrompts()
    clearForm()
  } catch (error) {
    console.error('Error al guardar el prompt:', error)
    // Aquí podrías mostrar un mensaje de error al usuario
  }
}

const loadSavedPrompts = async () => {
  try {
    const response = await axios.get(`${API_BASE_URL}/prompts`)
    savedPrompts.value = response.data
  } catch (error) {
    console.error('Error al cargar los prompts:', error)
    // Aquí podrías mostrar un mensaje de error al usuario
  }
}

const loadPrompt = (prompt) => {
  promptTitle.value = prompt.title
  promptText.value = prompt.promptText
  variables.value = { ...prompt.variables }
  currentPromptId.value = prompt._id
}

const deletePrompt = async (promptId) => {
  try {
    await axios.delete(`${API_BASE_URL}/prompts/${promptId}`)
    await loadSavedPrompts()
  } catch (error) {
    console.error('Error al eliminar el prompt:', error)
    // Aquí podrías mostrar un mensaje de error al usuario
  }
}

const clearForm = () => {
  promptTitle.value = ''
  promptText.value = ''
  variables.value = {}
  currentPromptId.value = null
}

onMounted(async () => {
  await loadSavedPrompts()
})
</script>