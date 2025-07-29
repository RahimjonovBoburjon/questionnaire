<template>
  <div class="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 flex items-center justify-center">
    <!-- Main Button -->
    <button @click="openModal"
      class="bg-indigo-600 hover:bg-indigo-700 text-white font-semibold py-3 px-8 rounded-lg shadow-lg transform transition-all duration-200 hover:scale-105 focus:outline-none focus:ring-4 focus:ring-indigo-300">
      Open Questionnaire
    </button>

    <!-- Modal Overlay -->
    <div v-if="isModalOpen"
      class="fixed inset-0 bg-black bg-opacity-50 backdrop-blur-sm flex items-center justify-center z-50"
      @click="closeModal">
      <!-- Modal Content -->
      <div class="bg-white rounded-xl shadow-2xl max-w-md w-full mx-4 max-h-[90vh] overflow-y-auto" @click.stop>
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

        let response
        try {
          response = await fetch(`https://api.telegram.org/bot${this.botToken}/sendMessage`, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
              chat_id: this.adminChatId,
              text: this.formatAnswers(answers)
            })
          })
        } catch (corsError) {
          console.log('Direct request failed due to CORS, trying with proxy...')
          const proxyUrl = 'https://cors-anywhere.herokuapp.com/'
          const telegramUrl = `https://api.telegram.org/bot${this.botToken}/sendMessage`

          response = await fetch(proxyUrl + telegramUrl, {
            method: 'POST',
            headers: {
              'Content-Type': 'application/json',
              'Origin': 'http://localhost:8080'
            },
            body: JSON.stringify({
              chat_id: this.adminChatId,
              text: this.formatAnswers(answers)
            })
          })
        }

        if (response.ok) {
          console.log('Answers sent successfully!')
        } else {
          console.error('Failed to send answers')
        }
      } catch (error) {
        console.error('Error sending answers:', error)
      }
    },
    formatAnswers(answers) {
      const questions = [
        'What is your favorite color?',
        'What is your age range?',
        'How did you hear about us?',
        'Would you recommend us?'
      ]

      let text = '📊 New questionnaire submitted:\n\n'
      answers.forEach((answer, index) => {
        text += `${index + 1}. ${questions[index]}\n   Answer: ${answer}\n\n`
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