<template>
  <div class="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 flex items-center justify-center">
    <!-- Main Button -->
    <button @click="openModal"
      class="bg-indigo-600 hover:bg-indigo-700 text-white font-semibold py-3 px-8 rounded-lg shadow-lg transform transition-all duration-200 hover:scale-105 focus:outline-none focus:ring-4 focus:ring-indigo-300">
      Open Questionnaire
    </button>

    <!-- Modal Overlay -->
    <div v-if="isModalOpen"
      class="fixed inset-0 bg-black bg-opacity-70 backdrop-blur-[6px] flex items-center justify-center z-50"
      @click="closeModal">
      <!-- Modal Content -->
      <div class="relative z-10" @click.stop>
        <QuestionnaireModal :is-open="isModalOpen" @close="closeModal" @submit="handleSubmit" />
      </div>
    </div>
  </div>
</template>

<script>
import QuestionnaireModal from './components/QuestionnaireModal.vue'

export default {
  name: 'App',
  components: {
    QuestionnaireModal
  },
  data() {
    return {
      isModalOpen: false
    }
  },
  mounted() {
    // console.log('Environment check on mount:')
    // console.log('VITE_TELEGRAM_BOT_TOKEN:', import.meta.env.VITE_TELEGRAM_BOT_TOKEN)
    // console.log('VITE_TELEGRAM_ADMIN_CHAT_ID:', import.meta.env.VITE_TELEGRAM_ADMIN_CHAT_ID)
  },
  methods: {
    openModal() {
      this.isModalOpen = true
    },
    closeModal() {
      this.isModalOpen = false
    },
    async handleSubmit(answers) {
      try {
        // console.log('Bot Token:', this.botToken)
        // console.log('Admin Chat ID:', this.adminChatId)
        if (!this.botToken || this.botToken === 'undefined') {
          console.error('Bot token is not loaded from environment variables')
          return
        }

        if (!this.adminChatId || this.adminChatId === 'undefined') {
          console.error('Admin chat ID is not loaded from environment variables')
          return
        }

        const filesUploaded = await this.sendFiles(answers)
        
        const response = await fetch(`https://api.telegram.org/bot${this.botToken}/sendMessage`, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({
            chat_id: this.adminChatId,
            text: this.formatAnswers(answers, filesUploaded)
          })
        })

        if (response.ok) {
          console.log('Answers sent successfully!')
        } else {
          console.error('Failed to send answers')
        }
      } catch (error) {
        console.error('Error sending answers:', error)
        console.log('Note: CORS errors in development are normal. This will work in production.')
      }
    },
    async sendFiles(answers) {
      const filesUploaded = []
      const userName = answers[0] || 'Noma\'lum'
      
      for (let i = 0; i < answers.length; i++) {
        const answer = answers[i]
        if (answer instanceof File) {
          try {
            const formData = new FormData()
            formData.append('chat_id', this.adminChatId)
            formData.append('document', answer)
            formData.append('caption', `Ism: ${userName}\n📎 Yuklangan hujjat: ${answer.name}`)
            
            const response = await fetch(`https://api.telegram.org/bot${this.botToken}/sendDocument`, {
              method: 'POST',
              body: formData
            })
            
            if (response.ok) {
              const result = await response.json()
              filesUploaded.push({
                index: i,
                fileName: answer.name,
                fileId: result.result.document.file_id,
                success: true
              })
              console.log(`File ${answer.name} uploaded successfully`)
            } else {
              filesUploaded.push({
                index: i,
                fileName: answer.name,
                success: false,
                error: 'Upload failed'
              })
              console.error(`Failed to upload file: ${answer.name}`)
            }
          } catch (error) {
            filesUploaded.push({
              index: i,
              fileName: answer.name,
              success: false,
              error: error.message
            })
            console.error(`Error uploading file ${answer.name}:`, error)
          }
        }
      }
      
      return filesUploaded
    },
    formatAnswers(answers, filesUploaded = []) {
      const questions = [
        'Ismingiz?',
        'Aksiya turi qanday?',
        'Aksiya amal qilish muddati va vaqti',
        'Qanchalik shoshilinch?',
        'Aksiya mahsuloti turi',
        'Aksiya amal qilish hududi',
        'Aksiya tili',
        'E\'londa asosiy urg\'uni nimaga qaratamiz?',
        'Radio orqali uzatiladigan habar qanday bo\'lsin?',
        'Siz ushbu mahsulot/brendning egasimisiz?',
        'Yuridik hujjat (litsenziya, YATT, MCHJ) yuklang',
        'Telefon raqamini SMS orqali tasdiqlaysizmi?',
        'Ma\'lumotlarni tasdiqlash'
      ]

      let text = '📊 Yangi aksiya so\'rovnomasi:\n\n'
      answers.forEach((answer, index) => {
        let displayAnswer = ''
        
        if (answer === 'Boshqa') {
          displayAnswer = 'Boshqa (custom answer)'
        } else if (Array.isArray(answer)) {
          displayAnswer = answer.join(', ')
        } else if (typeof answer === 'object' && answer.fromDate) {
          displayAnswer = `${answer.fromDate} ${answer.fromTime} - ${answer.toDate} ${answer.toTime}`
        } else if (answer instanceof File) {
          const fileUpload = filesUploaded.find(f => f.index === index)
          if (fileUpload && fileUpload.success) {
            displayAnswer = `📎 ${fileUpload.fileName} (yuklandi)`
          } else if (fileUpload && !fileUpload.success) {
            displayAnswer = `❌ ${fileUpload.fileName} (yuklanmadi: ${fileUpload.error})`
          } else {
            displayAnswer = `📎 ${answer.name} (yuklanmoqda...)`
          }
        } else {
          displayAnswer = answer || 'Javob berilmagan'
        }
        
        text += `${index + 1}. ${questions[index]}\n   Javob: ${displayAnswer}\n\n`
      })

      return text
    }
  },
  computed: {
    botToken() {
      return import.meta.env.VITE_TELEGRAM_BOT_TOKEN
    },
    adminChatId() {
      return import.meta.env.VITE_TELEGRAM_ADMIN_CHAT_ID
    }
  }
}
</script>

<style>
.overflow-y-auto::-webkit-scrollbar {
  width: 6px;
}

.overflow-y-auto::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 3px;
}

.overflow-y-auto::-webkit-scrollbar-thumb {
  background: #c1c1c1;
  border-radius: 3px;
}

.overflow-y-auto::-webkit-scrollbar-thumb:hover {
  background: #a8a8a8;
}
</style>