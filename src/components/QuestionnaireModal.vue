<template>
  <div class="flex flex-col items-center justify-center min-h-[80vh]">
    <!-- Logo -->
    <div class="z-20 flex justify-center" style="margin-bottom: -3.5rem;">
      <img src="/logo.png" alt="Logo" class="w-32 h-32 object-cover" style="object-position: center;" />
    </div>
    <!-- Card -->
    <div class="z-10 bg-white rounded-[2rem] shadow-xl w-[340px] max-w-full px-6 py-8 flex flex-col items-center">
      <!-- Intro Card -->
      <div v-if="currentStep === 0" class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-6 text-center">Sizni millionlar eshitadi va ko‘radi, hatto
          boshqa mamlakatlardan ham!</h3>
        <button @click="nextStep"
          class="bg-[#FF2D6A] hover:bg-[#e0265c] text-white font-semibold py-3 px-8 rounded-full w-full max-w-xs transition-colors duration-200">
          Boshlash
        </button>
      </div>
      <!-- Dynamic Question Cards -->
      <div v-else-if="currentStep > 0 && currentStep <= questions.length" class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-6 text-center">{{ questions[currentStep - 1].label }}</h3>
        <!-- Text Input -->
        <input v-if="questions[currentStep - 1].type === 'input'" v-model="answers[currentStep - 1]"
          :placeholder="questions[currentStep - 1].placeholder || ''" :class="getInputClass(currentStep - 1)"
          class="w-full rounded-xl border px-4 py-2 mb-4 focus:outline-none focus:ring-2" />
        <!-- Validation Error Message -->
        <div v-if="questions[currentStep - 1].type === 'input' && getValidationError(currentStep - 1)"
          class="text-red-500 text-sm mb-4">
          {{ getValidationError(currentStep - 1) }}
        </div>
        <!-- Radio/Checkbox Options -->
        <div v-if="['radio', 'checkbox'].includes(questions[currentStep - 1].type)"
          class="flex flex-col gap-3 w-full mb-4">
          <button v-for="option in questions[currentStep - 1].options" :key="option" @click="selectOption(option)"
            :class="optionButtonClass(option)">
            {{ option }}
          </button>
          <!-- Other logic -->
          <div v-if="questions[currentStep - 1].other && answers[currentStep - 1] === 'Boshqa'"
            class="w-full flex flex-col items-center">
            <input v-model="otherAnswers[currentStep - 1]" placeholder="Boshqa variantni kiriting"
              class="w-full rounded-xl border border-gray-200 px-4 py-2 mb-2 focus:outline-none focus:ring-2 focus:ring-[#FF2D6A]" />
            <button @click="nextStep" :disabled="!otherAnswers[currentStep - 1]"
              class="bg-[#FF2D6A] hover:bg-[#e0265c] text-white font-semibold py-3 px-8 rounded-full w-full max-w-xs transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed">
              OK
            </button>
          </div>
        </div>
        <!-- Multi-select Options -->
        <div v-if="questions[currentStep - 1].type === 'multiselect'" class="flex flex-col gap-3 w-full mb-4">
          <button v-for="option in questions[currentStep - 1].options" :key="option" @click="selectOption(option)"
            :class="optionButtonClass(option)">
            {{ option }}
          </button>
          <!-- Other logic for multiselect -->
          <div
            v-if="questions[currentStep - 1].other && answers[currentStep - 1] && answers[currentStep - 1].includes('Boshqa')"
            class="w-full flex flex-col items-center">
            <input v-model="otherAnswers[currentStep - 1]" placeholder="Boshqa variantni kiriting"
              class="w-full rounded-xl border border-gray-200 px-4 py-2 mb-2 focus:outline-none focus:ring-2 focus:ring-[#FF2D6A]" />
            <button @click="nextStep" :disabled="!otherAnswers[currentStep - 1]"
              class="bg-[#FF2D6A] hover:bg-[#e0265c] text-white font-semibold py-3 px-8 rounded-full w-full max-w-xs transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed">
              OK
            </button>
          </div>
        </div>
        <!-- Date Range -->
        <div v-if="questions[currentStep - 1].type === 'daterange'" class="w-full flex flex-col items-center mb-4">
          <div class="w-full mb-4">
            <label class="block text-sm font-medium text-gray-700 mb-2">Boshlanish sanasi va vaqti:</label>
            <input type="date" v-model="dateRange.fromDate" :class="getDateInputClass('from')"
              class="w-full rounded-xl border px-4 py-2 mb-2 focus:outline-none focus:ring-2" />
            <input type="time" v-model="dateRange.fromTime" :class="getDateInputClass('from')"
              class="w-full rounded-xl border px-4 py-2 mb-4 focus:outline-none focus:ring-2" />
          </div>
          <div class="w-full mb-4">
            <label class="block text-sm font-medium text-gray-700 mb-2">Tugash sanasi va vaqti:</label>
            <input type="date" v-model="dateRange.toDate" :class="getDateInputClass('to')"
              class="w-full rounded-xl border px-4 py-2 mb-2 focus:outline-none focus:ring-2" />
            <input type="time" v-model="dateRange.toTime" :class="getDateInputClass('to')"
              class="w-full rounded-xl border px-4 py-2 focus:outline-none focus:ring-2" />
          </div>
          <!-- Date Range Validation Error -->
          <div v-if="getDateRangeValidationError()" class="text-red-500 text-sm mb-4">
            {{ getDateRangeValidationError() }}
          </div>
        </div>
        <!-- File Upload -->
        <div v-if="questions[currentStep - 1].type === 'file'" class="w-full flex flex-col items-center">
          <input type="file" @change="onFileChange($event, currentStep - 1)" class="mb-4" />
          <!-- File Size Error Message -->
          <div v-if="getFileValidationError(currentStep - 1)" class="text-red-500 text-sm mb-4">
            {{ getFileValidationError(currentStep - 1) }}
          </div>
          <!-- File Info Display -->
          <div v-if="fileNames[currentStep - 1] && !getFileValidationError(currentStep - 1)"
            class="text-green-600 text-sm mb-4">
            ✅ {{ fileNames[currentStep - 1] }} ({{ getFileSizeDisplay(currentStep - 1) }})
          </div>
          <button @click="nextStep" :disabled="!isFileValid(currentStep - 1)"
            class="bg-[#FF2D6A] hover:bg-[#e0265c] text-white font-semibold py-3 px-8 rounded-full w-full max-w-xs transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed">
            Davom etish
          </button>
        </div>
        <!-- Default Next Button (if not Other) -->
        <button v-if="!isOtherActive(currentStep - 1) && questions[currentStep - 1].type !== 'file'" @click="nextStep"
          :disabled="!isStepValid(currentStep - 1)"
          class="bg-[#FF2D6A] hover:bg-[#e0265c] text-white font-semibold py-3 px-8 rounded-full w-full max-w-xs transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed">
          Davom etish
        </button>
        <button v-if="currentStep > 1" @click="previousStep"
          class="mt-2 w-full max-w-xs bg-gray-200 text-gray-700 font-semibold py-3 rounded-full transition-colors duration-200">
          Orqaga
        </button>
      </div>
      <!-- Summary/Confirmation -->
      <div v-else-if="currentStep === questions.length + 1" class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-6 text-center">Barcha ma’lumotlarni tasdiqlang</h3>
        <ul class="w-full mb-4">
          <li v-for="(q, idx) in questions" :key="q.label" class="mb-2 text-sm text-gray-700">
            <span class="font-semibold">{{ q.label }}:</span>
            <span v-if="q.other && answers[idx] === 'Boshqa'"> {{ otherAnswers[idx] }}</span>
            <span v-else-if="q.type === 'file'"> {{ fileNames[idx] || 'Fayl tanlanmagan' }}</span>
            <span v-else-if="q.type === 'daterange'">
              {{ answers[idx]?.fromDate }} {{ answers[idx]?.fromTime }} - {{ answers[idx]?.toDate }} {{
                answers[idx]?.toTime }}
            </span>
            <span v-else-if="q.type === 'multiselect'">
              {{ answers[idx]?.join(', ') }}
            </span>
            <span v-else> {{ answers[idx] }}</span>
          </li>
        </ul>
        <button @click="submitForm"
          class="bg-[#FF2D6A] hover:bg-[#e0265c] text-white font-semibold py-3 px-8 rounded-full w-full max-w-xs transition-colors duration-200">
          YUBORISH
        </button>
        <button @click="closeModal"
          class="mt-2 w-full max-w-xs bg-gray-200 text-gray-700 font-semibold py-3 rounded-full transition-colors duration-200">
          INKOR
        </button>
      </div>
      <!-- Completion Screen -->
      <div v-else class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-2 text-center">Thank you!</h3>
        <p class="text-gray-600 text-center mb-8">Thank you for participating in the survey.</p>
        <button @click="closeModal"
          class="w-full max-w-xs bg-[#FF2D6A] text-white font-semibold py-3 rounded-full transition-colors duration-200">
          Close
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'QuestionnaireModal',
  data() {
    return {
      currentStep: 0,
      questions: [
        { type: 'input', label: 'Ismingiz?', placeholder: 'Ismingizni kiriting' },
        {
          type: 'radio', label: 'Aksiya turi qanday?', options: [
            '1+1 aksiyasi', '2+1 aksiyasi', 'Diskont (narxlar pasayishi)', 'Rasprodaja (sotib tugatish)', '400 000 so‘mdan ko‘p xarid uchun sovg‘a', 'Har 10-mashinani yuvish bepul', 'Xarid uchun sovg‘a bor', 'Boshqa'
          ], other: true
        },
        { type: 'daterange', label: 'Aksiya amal qilish muddati va vaqti' },
        { type: 'radio', label: 'Qanchalik shoshilinch?', options: ['Shoshilinch emas', 'O‘rta shoshilinch', 'Juda shoshilinch'] },
        { type: 'radio', label: 'Aksiya mahsuloti turi', options: ['Xizmat ko‘rsatish / servis', 'Oziq-ovqat', 'Kiyim-kechak', 'Transport savdosi', 'Ko‘chmas mulk savdosi', 'Boshqa'], other: true },
        { type: 'checkbox', label: 'Aksiya amal qilish hududi', options: ['Yevropa', 'Osiyo', 'Afrika', 'Shimoliy Amerika', 'Janubiy Amerika', 'Avstraliya'] },
        { type: 'multiselect', label: 'Aksiya tili', options: ['O‘zbek', 'Ingliz', 'Turk', 'Fransuz', 'Nemis', 'Tojik', 'Hind', 'Ozarbayjon', 'Boshqa'], other: true },
        { type: 'radio', label: 'E\'londa asosiy urg‘uni nimaga qaratamiz?', options: ['Mahsulotga', 'Mahsulot soniga', 'Narxga', 'Sifatga', 'Aksiya shartiga', 'Brendga', 'Boshqa'], other: true },
        { type: 'radio', label: 'Radio orqali uzatiladigan habar qanday bo‘lsin?', options: ['Ha, audio rolik bo‘lsin', 'Yo‘q, boshlovchi og‘zaki e’lon qilsin', 'Farqi yo‘q'] },
        { type: 'radio', label: 'Siz ushbu mahsulot/brendning egasimisiz?', options: ['Ha, kompaniya rahbariman', 'Yo‘q, lekin vakolatim bor', 'Kampaniyaga aloqam yo‘q'] },
        { type: 'file', label: 'Yuridik hujjat (litsenziya, YATT, MCHJ) yuklang' },
        { type: 'radio', label: 'Telefon raqamini SMS orqali tasdiqlaysizmi?', options: ['Tasdiqlayman', 'Shart emas', 'Keyinroq'] },
        { type: 'radio', label: 'Ma’lumotlarni tasdiqlash', options: ['Ha', 'Yo‘q', 'Callback kutaman'] }
      ],
      answers: [],
      otherAnswers: [],
      fileNames: [],
      fileSizes: [],
      fileErrors: [],
      dateRange: {
        fromDate: '',
        fromTime: '',
        toDate: '',
        toTime: ''
      }
    }
  },
  methods: {
    nextStep() {
      const currentIdx = this.currentStep - 1
      if (this.questions[currentIdx] && this.questions[currentIdx].type === 'daterange') {
        this.answers[currentIdx] = {
          fromDate: this.dateRange.fromDate,
          fromTime: this.dateRange.fromTime,
          toDate: this.dateRange.toDate,
          toTime: this.dateRange.toTime
        }
      }

      this.currentStep++
      console.log(this.answers)
    },
    previousStep() {
      if (this.currentStep > 0) this.currentStep--
    },
    selectOption(option) {
      const idx = this.currentStep - 1
      const q = this.questions[idx]

      if (q.type === 'checkbox') {
        if (!Array.isArray(this.answers[idx])) this.answers[idx] = []
        const i = this.answers[idx].indexOf(option)
        if (i > -1) this.answers[idx].splice(i, 1)
        else this.answers[idx].push(option)
      } else if (q.type === 'multiselect') {
        if (!Array.isArray(this.answers[idx])) this.answers[idx] = []
        const i = this.answers[idx].indexOf(option)
        if (i > -1) this.answers[idx].splice(i, 1)
        else this.answers[idx].push(option)
      } else {
        this.answers[idx] = option
        if (this.questions[idx].other && option !== 'Boshqa') {
          this.otherAnswers[idx] = ''
        }
      }
    },
    isOtherActive(idx) {
      return this.questions[idx].other && this.answers[idx] === 'Boshqa'
    },
    isStepValid(idx) {
      const q = this.questions[idx]
      if (q.type === 'input') {
        if (!this.answers[idx]) return false
        if (idx === 0) {
          return this.validateName(this.answers[idx])
        }
        return true
      }
      if (q.type === 'radio') return !!this.answers[idx] && (q.other ? (this.answers[idx] !== 'Boshqa' || !!this.otherAnswers[idx]) : true)
      if (q.type === 'checkbox') return Array.isArray(this.answers[idx]) && this.answers[idx].length > 0
      if (q.type === 'multiselect') return Array.isArray(this.answers[idx]) && this.answers[idx].length > 0
      if (q.type === 'daterange') {
        return this.validateDateRange()
      }
      if (q.type === 'file') return this.isFileValid(idx)
      return true
    },
    validateName(name) {
      if (!name || typeof name !== 'string') return false
      if (name.trim().length < 3) return false
      const letterOnlyRegex = /^[\p{L}\s]+$/u
      return letterOnlyRegex.test(name.trim())
    },
    validateDateRange() {
      if (!this.dateRange.fromDate || !this.dateRange.fromTime || !this.dateRange.toDate || !this.dateRange.toTime) {
        return false
      }

      const fromYear = new Date(this.dateRange.fromDate).getFullYear()
      const toYear = new Date(this.dateRange.toDate).getFullYear()
      if (fromYear > 2400 || toYear > 2400) return false

      const startDateTime = new Date(`${this.dateRange.fromDate}T${this.dateRange.fromTime}`)
      const endDateTime = new Date(`${this.dateRange.toDate}T${this.dateRange.toTime}`)

      return endDateTime > startDateTime
    },
    getValidationError(idx) {
      const q = this.questions[idx]
      if (q.type === 'input' && idx === 0) {
        const name = this.answers[idx]
        if (!name) return ''
        if (name.trim().length < 3) return 'Ism kamida 3 ta harf bo\'lishi kerak'
        const letterOnlyRegex = /^[\p{L}\s]+$/u
        if (!letterOnlyRegex.test(name.trim())) return 'Ism faqat harflardan iborat bo\'lishi kerak'
      }
      return ''
    },
    getDateRangeValidationError() {
      if (!this.dateRange.fromDate || !this.dateRange.fromTime || !this.dateRange.toDate || !this.dateRange.toTime) {
        return ''
      }

      const fromYear = new Date(this.dateRange.fromDate).getFullYear()
      const toYear = new Date(this.dateRange.toDate).getFullYear()
      if (fromYear > 2400 || toYear > 2400) {
        return 'Yil 2400 dan oshmasligi kerak'
      }

      const startDateTime = new Date(`${this.dateRange.fromDate}T${this.dateRange.fromTime}`)
      const endDateTime = new Date(`${this.dateRange.toDate}T${this.dateRange.toTime}`)

      if (endDateTime <= startDateTime) {
        return 'Tugash vaqti boshlanish vaqtidan keyin bo\'lishi kerak'
      }

      return ''
    },
    getInputClass(idx) {
      const hasError = this.getValidationError(idx)
      return hasError ? 'border-red-500 focus:ring-red-500' : 'border-gray-200 focus:ring-[#FF2D6A]'
    },
    getDateInputClass(type) {
      const hasError = this.getDateRangeValidationError()
      return hasError ? 'border-red-500 focus:ring-red-500' : 'border-gray-200 focus:ring-[#FF2D6A]'
    },
    optionButtonClass(option) {
      const idx = this.currentStep - 1
      const q = this.questions[idx]
      if (q.type === 'checkbox' || q.type === 'multiselect') {
        return [
          'bg-gray-100 text-gray-700 font-medium py-3 rounded-full w-full transition-all duration-150 focus:outline-none focus:ring-2 focus:ring-[#FF2D6A] border-2',
          (Array.isArray(this.answers[idx]) && this.answers[idx].includes(option)) ? 'border-[#FF2D6A] bg-[#FFF0F6]' : 'border-gray-100'
        ]
      } else {
        return [
          'bg-gray-100 text-gray-700 font-medium py-3 rounded-full w-full transition-all duration-150 focus:outline-none focus:ring-2 focus:ring-[#FF2D6A] border-2',
          (this.answers[idx] === option) ? 'border-[#FF2D6A] bg-[#FFF0F6]' : 'border-gray-100'
        ]
      }
    },
    onFileChange(e, idx) {
      const file = e.target.files[0]
      if (file) {
        const maxSizeBytes = 50 * 1024 * 1024

        if (file.size > maxSizeBytes) {
          this.fileNames[idx] = file.name
          this.fileSizes[idx] = file.size
          this.fileErrors[idx] = 'Fayl hajmi 50 MB dan oshmasligi kerak'
          this.answers[idx] = null
        } else {
          this.fileNames[idx] = file.name
          this.fileSizes[idx] = file.size
          this.fileErrors[idx] = null
          this.answers[idx] = file
        }
      } else {
        this.fileNames[idx] = null
        this.fileSizes[idx] = null
        this.fileErrors[idx] = null
        this.answers[idx] = null
      }
    },
    isFileValid(idx) {
      return !!this.fileNames[idx] && !this.fileErrors[idx] && !!this.answers[idx]
    },
    getFileValidationError(idx) {
      return this.fileErrors[idx] || ''
    },
    getFileSizeDisplay(idx) {
      if (!this.fileSizes[idx]) return ''
      const sizeInMB = (this.fileSizes[idx] / (1024 * 1024)).toFixed(2)
      return `${sizeInMB} MB`
    },
    submitForm() {
      this.$emit('submit', this.answers)
      this.currentStep++
    },
    closeModal() {
      this.currentStep = 0
      this.answers = []
      this.otherAnswers = []
      this.fileNames = []
      this.fileSizes = []
      this.fileErrors = []
      this.dateRange = {
        fromDate: '',
        fromTime: '',
        toDate: '',
        toTime: ''
      }
      this.$emit('close')
    }
  }
}
</script>

<style scoped>
body {
  background: #3a393a;
}
</style>