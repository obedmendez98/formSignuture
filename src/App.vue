<template>
  <div class="max-w-4xl mx-auto p-4">
    <!-- Form -->
    <form
      @submit.prevent="handleSubmit()"
      class="bg-white rounded-lg shadow-md p-6"
    >
      <div v-for="field in formConfig" :key="field.name" class="mb-6">
        <label :for="field.name" class="block text-sm font-medium text-gray-700 mb-2">
          {{ field.label }}
          <span v-if="field.required" class="text-red-500">*</span>
        </label>

        <!-- Text or Email Input -->
        <div v-if="['text', 'email'].includes(field.type)" class="relative">
          <input
            :type="field.type"
            :id="field.name"
            :name="field.name"
            v-model="formData[field.name]"
            @blur="validateField(field.name)"
            :placeholder="field.placeholder || ''"
            class="w-full px-4 py-2 border rounded-md focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition-colors"
            :class="{'border-red-500': errors[field.name], 'border-gray-300': !errors[field.name]}"
          />
          <span v-if="errors[field.name]" class="text-red-500 text-sm mt-1 block">
            {{ errors[field.name] }}
          </span>
        </div>

        <!-- Textarea -->
        <div v-else-if="field.type === 'textarea'" class="relative">
          <textarea
            :id="field.name"
            :name="field.name"
            v-model="formData[field.name]"
            @blur="validateField(field.name)"
            :placeholder="field.placeholder || ''"
            :rows="field.rows || 4"
            class="w-full px-4 py-2 border rounded-md focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition-colors"
            :class="{'border-red-500': errors[field.name], 'border-gray-300': !errors[field.name]}"
          ></textarea>
          <span v-if="errors[field.name]" class="text-red-500 text-sm mt-1 block">
            {{ errors[field.name] }}
          </span>
        </div>

        <!-- Checkbox -->
        <div v-else-if="field.type === 'checkbox'" class="flex items-center gap-2">
          <input
            type="checkbox"
            :id="field.name"
            :name="field.name"
            v-model="formData[field.name]"
            class="w-4 h-4 text-blue-600 border-gray-300 rounded focus:ring-blue-500"
          />
          <span class="text-sm text-gray-700">{{ field.label }}</span>
        </div>

        <!-- Date Input -->
        <div v-else-if="field.type === 'date'" class="relative">
          <input
            type="date"
            :id="field.name"
            :name="field.name"
            v-model="formData[field.name]"
            @blur="validateField(field.name)"
            class="w-full px-4 py-2 border rounded-md focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition-colors"
            :class="{'border-red-500': errors[field.name], 'border-gray-300': !errors[field.name]}"
          />
          <span v-if="errors[field.name]" class="text-red-500 text-sm mt-1 block">
            {{ errors[field.name] }}
          </span>
        </div>

        <!-- Time Input -->
        <div v-else-if="field.type === 'time'" class="relative">
          <input
            type="time"
            :id="field.name"
            :name="field.name"
            v-model="formData[field.name]"
            @blur="validateField(field.name)"
            class="w-full px-4 py-2 border rounded-md focus:ring-2 focus:ring-blue-500 focus:border-blue-500 transition-colors"
            :class="{'border-red-500': errors[field.name], 'border-gray-300': !errors[field.name]}"
          />
          <span v-if="errors[field.name]" class="text-red-500 text-sm mt-1 block">
            {{ errors[field.name] }}
          </span>
        </div>

        <!-- Switch -->
        <div v-else-if="field.type === 'switch'" class="flex items-center gap-2">
          <input
            type="checkbox"
            :id="field.name"
            :name="field.name"
            v-model="formData[field.name]"
            class="w-5 h-5 cursor-pointer transition-colors"
          />
          <span class="text-sm text-gray-700">{{ field.label }}</span>
        </div>

      </div>

      <!-- Signature Canvas -->
      <div class="mb-6">
        <label class="block text-sm font-medium text-gray-700 mb-2">
          Signature
          <span class="text-red-500">*</span>
        </label>
        <div class="relative" ref="canvasContainer">
          <canvas
            ref="signatureRef"
            class="border border-gray-300 rounded-md w-full h-40 cursor-crosshair bg-gray-50"
            @mousedown="startDrawing"
            @mousemove="draw"
            @mouseup="stopDrawing"
            @mouseleave="stopDrawing"
            @touchstart.prevent="handleTouchStart"
            @touchmove.prevent="handleTouchMove"
            @touchend.prevent="stopDrawing"
          ></canvas>
          <div class="flex justify-between mt-2">
            <button 
              type="button" 
              @click="clearSignature"
              class="px-4 py-2 text-sm text-gray-600 hover:text-gray-800 transition-colors"
            >
              Clear Signature
            </button>
            <span v-if="!hasSignature && showSignatureError" class="text-red-500 text-sm">
              Signature is required
            </span>
          </div>
        </div>
      </div>

      <!-- Action Buttons -->
      <div class="flex justify-end gap-4">
        <button
          type="button"
          @click="cancelForm"
          class="px-6 py-2 border border-gray-300 rounded-md text-gray-700 hover:bg-gray-50 transition-colors"
        >
          Cancel
        </button>
        <button
          type="submit"
          class="px-6 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700 transition-colors disabled:opacity-50 disabled:cursor-not-allowed"
        >
          Complete Purchase
        </button>
      </div>
    </form>
  </div>
</template>

<script>
import { ref, reactive, nextTick } from 'vue';

export default {
  name: 'App',
  setup() {
    const showSuccessMessage = ref(false);
    const formData = reactive({});
    const errors = reactive({});
    const isDrawing = ref(false);
    const signatureRef = ref(null);
    const canvasContainer = ref(null);
    const hasSignature = ref(false);
    const showSignatureError = ref(false);
    const canvasInitialized = ref(false);

    return {
      showSuccessMessage,
      formData,
      errors,
      isDrawing,
      signatureRef,
      canvasContainer,
      hasSignature,
      showSignatureError,
      canvasInitialized
    };
  },
  data() {
    return {
      formConfig: [
    { 
      name: 'name', 
      label: 'Full Name', 
      type: 'text', 
      required: true, 
      minLength: 3, 
      maxLength: 50,
      placeholder: 'Enter your full name'
    },
    { 
      name: 'email', 
      label: 'Email Address', 
      type: 'email', 
      required: true,
      placeholder: 'example@email.com'
    },
    { 
      name: 'address', 
      label: 'Shipping Address', 
      type: 'textarea', 
      required: true, 
      minLength: 10, 
      maxLength: 500,
      placeholder: 'Enter your complete address'
    },
    { 
      name: 'terms', 
      label: 'I accept the terms and conditions', 
      type: 'checkbox',
      required: true
    },
    { 
      name: 'birthdate', 
      label: 'Date of Birth', 
      type: 'date', 
      required: true
    },
    { 
      name: 'appointmentTime', 
      label: 'Appointment Time', 
      type: 'time', 
      required: true
    },
    { 
      name: 'subscribe', 
      label: 'Subscribe to the newsletter', 
      type: 'switch', 
      required: false 
    }
],

      signatureContext: null,
      lastX: 0,
      lastY: 0,
      resizeObserver: null,
      canvasRetries: 0,
      maxCanvasRetries: 3
    };
  },
  computed: {
    isFormValid() {
    console.log("Checking form validity...");
    console.log("Errors:", this.errors);
    console.log("Form data:", this.formData);
    return Object.keys(this.errors).length === 0 && 
      this.formConfig.every(field => {
        const isValid = !field.required || this.formData[field.name];
        console.log(`Field ${field.name} valid:`, isValid);
        return isValid;
      });
    } 
  },
  methods: {

    async initCanvas() {
      //console.log('Iniciando canvas...');
      
      if (this.canvasRetries >= this.maxCanvasRetries) {
        console.error('Max canvas initialization retries reached');
        return;
      }

      await nextTick();
      
      const canvas = this.$refs.signatureRef;
      const container = this.$refs.canvasContainer;
      
      if (!canvas || !container) {
        //console.log('Canvas o container no encontrados, reintentando...');
        this.canvasRetries++;
        setTimeout(() => this.initCanvas(), 100);
        return;
      }

      // Obtener dimensiones del contenedor
      const containerWidth = container.offsetWidth;
      if (containerWidth === 0) {
        //console.log('Contenedor con ancho 0, reintentando...');
        this.canvasRetries++;
        setTimeout(() => this.initCanvas(), 100);
        return;
      }

      // Configurar canvas
      canvas.width = containerWidth;
      canvas.height = 160; // height-40 en píxeles

      this.signatureContext = canvas.getContext('2d');
      if (!this.signatureContext) {
        console.error('error canvas');
        return;
      }

      // Configurar contexto
      this.signatureContext.strokeStyle = '#000';
      this.signatureContext.lineWidth = 2;
      this.signatureContext.lineCap = 'round';
      this.signatureContext.lineJoin = 'round';

      this.canvasInitialized = true;
      //console.log('Canvas inicializado correctamente');

      // Configurar ResizeObserver
      if (!this.resizeObserver) {
        this.resizeObserver = new ResizeObserver(() => {
          if (this.canvasInitialized) {
            this.initCanvas();
          }
        });
        this.resizeObserver.observe(container);
      }
    },

    validateField(fieldName) {
      console.log(fieldName)
      const field = this.formConfig.find(f => f.name === fieldName);
      console.log(field);
      if (!field) return;

      delete this.errors[fieldName];
      console.log("1")

      if (field.required && !this.formData[fieldName]) {
        console.log("2")

        this.errors[fieldName] = 'Este campo es requerido';
      }

      if (field.type === 'email' && this.formData[fieldName]) {
        console.log("3")

        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        if (!emailRegex.test(this.formData[fieldName])) {
          console.log("4")

          this.errors[fieldName] = 'Por favor ingrese un email válido';
          
        }
      }

      if (field.minLength && this.formData[fieldName].length < field.minLength) {
        console.log("5")

        this.errors[fieldName] = `Mínimo ${field.minLength} caracteres`;
        console.log(this.errors[fieldName])
      }

      if (field.maxLength && this.formData[fieldName].length > field.maxLength) {
        console.log("6")

        this.errors[fieldName] = `Máximo ${field.maxLength} caracteres`;
      }

      console.log(this.errors[fieldName])
    },

    handleTouchStart(event) {
      if (!this.signatureContext || !this.canvasInitialized) return;
      
      const touch = event.touches[0];
      const rect = this.$refs.signatureRef.getBoundingClientRect();
      
      this.isDrawing = true;
      this.lastX = touch.clientX - rect.left;
      this.lastY = touch.clientY - rect.top;
      this.hasSignature = true;
    },

    handleTouchMove(event) {
      if (!this.isDrawing || !this.signatureContext || !this.canvasInitialized) return;

      const touch = event.touches[0];
      const rect = this.$refs.signatureRef.getBoundingClientRect();
      
      const currentX = touch.clientX - rect.left;
      const currentY = touch.clientY - rect.top;

      this.signatureContext.beginPath();
      this.signatureContext.moveTo(this.lastX, this.lastY);
      this.signatureContext.lineTo(currentX, currentY);
      this.signatureContext.stroke();

      this.lastX = currentX;
      this.lastY = currentY;
    },

    startDrawing(event) {
      if (!this.signatureContext || !this.canvasInitialized) return;
      
      const rect = this.$refs.signatureRef.getBoundingClientRect();
      
      this.isDrawing = true;
      this.lastX = event.clientX - rect.left;
      this.lastY = event.clientY - rect.top;
      this.hasSignature = true;
      this.showSignatureError = false;
    },

    draw(event) {
      if (!this.isDrawing || !this.signatureContext || !this.canvasInitialized) return;

      const rect = this.$refs.signatureRef.getBoundingClientRect();
      const currentX = event.clientX - rect.left;
      const currentY = event.clientY - rect.top;

      this.signatureContext.beginPath();
      this.signatureContext.moveTo(this.lastX, this.lastY);
      this.signatureContext.lineTo(currentX, currentY);
      this.signatureContext.stroke();

      this.lastX = currentX;
      this.lastY = currentY;
    },

    stopDrawing() {
      this.isDrawing = false;
    },

    clearSignature() {
      if (this.signatureContext && this.$refs.signatureRef) {
        this.signatureContext.clearRect(
          0, 
          0, 
          this.$refs.signatureRef.width, 
          this.$refs.signatureRef.height
        );
        this.hasSignature = false;
      }
    },

    getSignatureData() {
      return this.$refs.signatureRef?.toDataURL() || null;
    },

    validateForm() {
      this.formConfig.forEach(field => {
        this.validateField(field.name);
      });

      console.log(this.hasSignature , "   dddd")
      if (!this.hasSignature) {
        this.showSignatureError = true;
        return false;
      }

      console.log(this.isFormValid, "  2")
      console.log(this.hasSignature, "  3")
      return this.isFormValid && this.hasSignature;
    },

    cancelForm() {
      this.resetForm();
    },
  resetForm() {
      Object.keys(this.formData).forEach(key => {
        this.formData[key] = this.formConfig.find(f => f.name === key).type === 'checkbox' || this.formConfig.find(f => f.name === key).type === 'switch' ? false : '';
      });
      this.clearSignature();
      this.hasSignature = false;
      this.showSignatureError = false;
      Object.keys(this.errors).forEach(key => delete this.errors[key]);
    },

    handleSubmit() {
  //console.log("Iniciando submit...");
  
  // Verificar el estado de la validación
  const isValid = this.validateForm();
  //console.log("¿Formulario válido?:", isValid);
  
  if (!isValid) {
    console.log('Formulario inválido');
    console.log('Errores:', this.errors);
    console.log('Estado de firma:', this.hasSignature);
    return;
  }
  
  const signatureData = this.getSignatureData();
  const submitData = {
    ...this.formData,
    signature: signatureData,
  };
  
  console.log('form:', submitData);
  
  // Mostrar mensaje de éxito
  this.showSuccessMessage = true;
}
  },
  
  async mounted() {
    //console.log("Componente montado");
    // Inicializar formData
    this.formConfig.forEach(field => {
      this.formData[field.name] = field.type === 'checkbox' || field.type === 'switch' ? false : '';
    });

    // Solo inicializar el canvas si el formulario está visible
    await this.initCanvas();

  },

  beforeUnmount() {
    // Limpiar el ResizeObserver
    if (this.resizeObserver) {
      this.resizeObserver.disconnect();
    }
  },
};
</script>

<style scoped>
@tailwind base;
@tailwind components;
@tailwind utilities;

canvas {
  touch-action: none; /* Mejora el manejo táctil */
}


</style>