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
    formatAnswers(formData, filesUploaded = []) {
      let text = '📊 Radio Yonar - Yangi biznes so\'rovnomasi:\n\n'
      
      // Format the new Radio Yonar form data
      text += `👤 Ism: ${formData.name || 'Kiritilmagan'}\n`
      text += `✅ Vakillik tasdiqlangan: ${formData.isRepresentative ? 'Ha' : 'Yo\'q'}\n`
      text += `✅ Ma\'lumotlar aniqligi tasdiqlangan: ${formData.confirmAccuracy ? 'Ha' : 'Yo\'q'}\n\n`
      
      text += `📝 Aksiya tavsifi:\n${formData.promotionDescription || 'Kiritilmagan'}\n\n`
      
      text += `🌍 Geografiya: ${formData.geography || 'Kiritilmagan'}\n\n`
      
      if (formData.promotionLanguages && formData.promotionLanguages.length > 0) {
        text += `🗣️ Aksiya tillari: ${formData.promotionLanguages.join(', ')}\n\n`
      }
      
      const location = formData.manualAddress || formData.location
      if (location) {
        text += `📍 Joylashuv: ${location}\n\n`
      }
      
      text += `📞 Kontakt ma\'lumotlari:\n`
      text += `   Mobil: ${formData.mobilePhone || 'Kiritilmagan'}\n`
      if (formData.officePhone) {
        text += `   Ish telefoni: ${formData.officePhone}\n`
      }
      text += `   Email: ${formData.email || 'Kiritilmagan'}\n`
      text += `   Qayta qo\'ng\'iroq ruxsati: ${formData.allowCallback ? 'Ha' : 'Yo\'q'}\n\n`
      
      // Handle file uploads
      const fileFields = [
        { key: 'businessPhoto', label: 'Biznes fotosi' },
        { key: 'documentPhoto', label: 'Hujjat fotosi' },
        { key: 'selfieVideo', label: 'Selfi video' }
      ]
      
      fileFields.forEach(field => {
        if (formData[field.key]) {
          const fileUpload = filesUploaded.find(f => f.fieldName === field.key)
          if (fileUpload && fileUpload.success) {
            text += `📎 ${field.label}: ${fileUpload.fileName} (yuklandi)\n`
          } else if (fileUpload && !fileUpload.success) {
            text += `❌ ${field.label}: ${fileUpload.fileName} (yuklanmadi: ${fileUpload.error})\n`
          } else {
            text += `📎 ${field.label}: ${formData[field.key].name} (yuklanmoqda...)\n`
          }
        }
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