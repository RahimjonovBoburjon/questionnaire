<template>
  <div class="flex flex-col items-center justify-start w-full">
    <!-- Logo -->
    <div class="z-20 flex justify-center flex-shrink-0" style="margin-bottom: -3.5rem;">
      <img src="/logo.png" alt="Logo" class="w-32 h-32 object-cover" style="object-position: center;" />
    </div>
    <!-- Card -->
    <div class="z-10 bg-white rounded-[2rem] shadow-xl w-full max-w-[400px] px-6 py-8 flex flex-col items-center">
      <!-- Language Selection -->
      <div v-if="currentStep === 0" class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-2 text-center">{{ getTitle('language') }}</h3>
        <p class="text-sm text-gray-600 mb-6 text-center">{{ getSubtitle('language') }}</p>

        <div class="w-full mb-6">
          <select v-model="selectedLanguage"
            class="w-full rounded-xl border border-gray-200 px-4 py-3 focus:outline-none focus:ring-2 focus:ring-[#FF2D6A] text-center">
            <option value="">{{ getPlaceholder('selectLanguage') }}</option>
            <option value="uz">O'ZBEKCHA</option>
            <option value="ru">РУССКИЙ</option>
            <option value="en">ENGLISH</option>
          </select>
        </div>

        <button @click="setLanguageAndContinue" :disabled="!selectedLanguage"
          class="bg-[#FF2D6A] hover:bg-[#e0265c] text-white font-semibold py-3 px-8 rounded-full w-full max-w-xs transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed">
          OK
        </button>
      </div>

      <!-- Terms & Conditions -->
      <div v-else-if="currentStep === 1" class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-4 text-center">{{ getText('termsTitle') }}</h3>

        <div class="w-full mb-6 max-h-[40vh] overflow-y-auto bg-gray-50 rounded-xl p-4 text-sm text-gray-700">
          <div class="mb-4">
            <h4 class="font-semibold mb-2">{{ getText('responsibilityTitle') }}</h4>
            <p class="mb-2">{{ getText('responsibilityText') }}</p>
            <ul class="list-disc list-inside space-y-1 ml-2">
              <li>{{ getText('responsibility1') }}</li>
              <li>{{ getText('responsibility2') }}</li>
              <li>{{ getText('responsibility3') }}</li>
            </ul>
          </div>

          <div class="mb-4">
            <h4 class="font-semibold mb-2">{{ getText('publicityTitle') }}</h4>
            <p class="mb-2">{{ getText('publicityText') }}</p>
            <ul class="list-disc list-inside space-y-1 ml-2">
              <li>{{ getText('publicity1') }}</li>
              <li>{{ getText('publicity2') }}</li>
              <li>{{ getText('publicity3') }}</li>
            </ul>
          </div>

          <div>
            <h4 class="font-semibold mb-2">{{ getText('consentTitle') }}</h4>
            <p>{{ getText('consentText') }}</p>
          </div>
        </div>

        <button @click="nextStep"
          class="bg-[#FF2D6A] hover:bg-[#e0265c] text-white font-semibold py-3 px-8 rounded-full w-full max-w-xs transition-colors duration-200">
          {{ getText('agreeButton') }}
        </button>
      </div>
      <!-- Step 1: Name & Checkboxes (1/10) -->
      <div v-else-if="currentStep === 2" class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-2 text-center">{{ getText('nameTitle') }} 1/10</h3>

        <input v-model="formData.name" :placeholder="getText('namePlaceholder')"
          class="w-full rounded-xl border border-gray-200 px-4 py-3 mb-2 focus:outline-none focus:ring-2 focus:ring-[#FF2D6A]" />

        <div v-if="formData.name && !isValidName(formData.name)" class="text-red-500 text-sm mb-4">
          {{ getText('nameError') || 'Имя должно содержать минимум 3 буквы и только буквы' }}
        </div>

        <div class="w-full space-y-3 mb-6">
          <label class="flex items-start space-x-3 cursor-pointer">
            <input type="checkbox" v-model="formData.isRepresentative"
              class="mt-1 rounded border-gray-300 text-[#FF2D6A] focus:ring-[#FF2D6A]">
            <span class="text-sm text-gray-700">{{ getText('representativeCheck') }}</span>
          </label>

          <label class="flex items-start space-x-3 cursor-pointer">
            <input type="checkbox" v-model="formData.confirmAccuracy"
              class="mt-1 rounded border-gray-300 text-[#FF2D6A] focus:ring-[#FF2D6A]">
            <span class="text-sm text-gray-700">{{ getText('accuracyCheck') }}</span>
          </label>
        </div>

        <button @click="nextStep" :disabled="!canProceed()"
          class="bg-[#FF2D6A] hover:bg-[#e0265c] text-white font-semibold py-3 px-8 rounded-full w-full max-w-xs transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed">
          {{ getText('continueButton') }}
        </button>
      </div>

      <!-- Step 2: Promotion Description (2/10) -->
      <div v-else-if="currentStep === 3" class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-2 text-center">{{ getText('promotionTitle') }} 2/10</h3>

        <div class="w-full mb-4 bg-gray-50 rounded-xl p-4">
          <p class="text-sm font-semibold text-gray-700 mb-2">{{ getText('examplesTitle') }}</p>
          <ul class="text-xs text-gray-600 space-y-1">
            <li>• {{ getText('example1') }}</li>
            <li>• {{ getText('example2') }}</li>
            <li>• {{ getText('example3') }}</li>
            <li>• {{ getText('example4') }}</li>
            <li>• {{ getText('example5') }}</li>
          </ul>
        </div>

        <textarea v-model="formData.promotionDescription" :placeholder="getText('promotionPlaceholder')" rows="6"
          class="w-full rounded-xl border border-gray-200 px-4 py-3 mb-4 focus:outline-none focus:ring-2 focus:ring-[#FF2D6A] resize-none"></textarea>

        <button @click="nextStep" :disabled="!formData.promotionDescription?.trim()"
          class="bg-[#FF2D6A] hover:bg-[#e0265c] text-white font-semibold py-3 px-8 rounded-full w-full max-w-xs transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed">
          {{ getText('continueButton') }}
        </button>
      </div>

      <!-- Step 3: Geography (3/10) -->
      <div v-else-if="currentStep === 4" class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-2 text-center">{{ getText('geographyTitle') }} 3/10</h3>
        <p class="text-sm text-gray-600 mb-6 text-center">{{ getText('geographySubtitle') }}</p>

        <input v-model="formData.geography" :placeholder="getText('geographyPlaceholder')"
          class="w-full rounded-xl border border-gray-200 px-4 py-3 mb-6 focus:outline-none focus:ring-2 focus:ring-[#FF2D6A]" />

        <button @click="nextStep" :disabled="!formData.geography?.trim()"
          class="bg-[#FF2D6A] hover:bg-[#e0265c] text-white font-semibold py-3 px-8 rounded-full w-full max-w-xs transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed">
          {{ getText('continueButton') }}
        </button>
      </div>

      <!-- Step 5: Promotion Languages (4/10) -->
      <div v-else-if="currentStep === 5" class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-2 text-center">{{ getText('languagesTitle') || 'ЯЗЫКИ АКЦИИ' }} 4/10</h3>
        <p class="text-sm text-gray-600 mb-6 text-center">{{ getText('languagesSubtitle') || 'На каких языках будет проводиться акция?' }}</p>
        <div class="w-full mb-6 space-y-3">
          <label v-for="lang in ['Узбекский', 'Русский', 'Английский', 'Турецкий', 'Другой']" :key="lang" 
            class="flex items-center space-x-3 cursor-pointer">
            <input type="checkbox" :value="lang" v-model="formData.promotionLanguages"
              class="w-5 h-5 text-[#FF2D6A] border-gray-300 rounded focus:ring-[#FF2D6A]">
            <span class="text-gray-700">{{ lang }}</span>
          </label>
        </div>
        <div class="flex gap-3 w-full max-w-xs">
          <button @click="prevStep" 
            class="bg-gray-200 hover:bg-gray-300 text-gray-700 font-semibold py-3 px-6 rounded-full flex-1 transition-colors duration-200">
            {{ getText('backButton') || 'Назад' }}
          </button>
          <button @click="nextStep" :disabled="!canProceed()"
            class="bg-[#FF2D6A] hover:bg-[#e0265c] text-white font-semibold py-3 px-6 rounded-full flex-1 transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed">
            {{ getText('continueButton') }}
          </button>
        </div>
      </div>

      <!-- Step 6: Location (5/10) -->
      <div v-else-if="currentStep === 6" class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-2 text-center">{{ getText('locationTitle') || 'МЕСТОПОЛОЖЕНИЕ' }} 5/10</h3>
        <p class="text-sm text-gray-600 mb-6 text-center">{{ getText('locationSubtitle') || 'Укажите адрес или выберите на карте' }}</p>
        
        <!-- Map placeholder -->
        <div class="w-full mb-4 h-40 bg-gray-100 rounded-xl flex items-center justify-center border-2 border-dashed border-gray-300">
          <span class="text-gray-500">{{ getText('mapPlaceholder') || 'Карта (будет добавлена позже)' }}</span>
        </div>
        
        <div class="w-full mb-6">
          <input v-model="formData.manualAddress" 
            :placeholder="getText('addressPlaceholder') || 'Или введите адрес вручную'"
            class="w-full rounded-xl border border-gray-200 px-4 py-3 focus:outline-none focus:ring-2 focus:ring-[#FF2D6A]">
        </div>
        <div class="flex gap-3 w-full max-w-xs">
          <button @click="prevStep" 
            class="bg-gray-200 hover:bg-gray-300 text-gray-700 font-semibold py-3 px-6 rounded-full flex-1 transition-colors duration-200">
            {{ getText('backButton') || 'Назад' }}
          </button>
          <button @click="nextStep" :disabled="!canProceed()"
            class="bg-[#FF2D6A] hover:bg-[#e0265c] text-white font-semibold py-3 px-6 rounded-full flex-1 transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed">
            {{ getText('continueButton') }}
          </button>
        </div>
      </div>

      <!-- Step 7: Photo/Video Uploads (6/10) -->
      <div v-else-if="currentStep === 7" class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-2 text-center">{{ getText('uploadsTitle') || 'ФОТО И ВИДЕО' }} 6/10</h3>
        <p class="text-sm text-gray-600 mb-6 text-center">{{ getText('uploadsSubtitle') || 'Загрузите фото бизнеса и документы' }}</p>
        
        <!-- Business Photo Upload -->
        <div class="w-full mb-4">
          <label class="block text-sm font-medium text-gray-700 mb-2">{{ getText('businessPhotoLabel') || 'Фото бизнеса' }}</label>
          <div class="relative">
            <input type="file" @change="handleFileUpload($event, 'businessPhoto')" accept="image/*,video/*"
              class="absolute inset-0 w-full h-full opacity-0 cursor-pointer">
            <div class="w-full h-32 bg-gray-50 border-2 border-dashed border-gray-300 rounded-xl flex flex-col items-center justify-center hover:bg-gray-100 transition-colors">
              <svg class="w-8 h-8 text-gray-400 mb-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path>
              </svg>
              <span class="text-sm text-gray-500">{{ getText('uploadBusinessPhoto') || 'Загрузить фото бизнеса' }}</span>
            </div>
          </div>
          <div v-if="formData.businessPhoto" class="mt-2 text-sm text-green-600">
            ✓ {{ formData.businessPhoto.name }}
          </div>
          <div v-if="fileErrors.businessPhoto" class="mt-2 text-sm text-red-600">
            {{ fileErrors.businessPhoto }}
          </div>
        </div>

        <!-- Document Photo Upload -->
        <div class="w-full mb-6">
          <label class="block text-sm font-medium text-gray-700 mb-2">{{ getText('documentPhotoLabel') || 'Документы' }}</label>
          <div class="relative">
            <input type="file" @change="handleFileUpload($event, 'documentPhoto')" accept="image/*"
              class="absolute inset-0 w-full h-full opacity-0 cursor-pointer">
            <div class="w-full h-32 bg-gray-50 border-2 border-dashed border-gray-300 rounded-xl flex flex-col items-center justify-center hover:bg-gray-100 transition-colors">
              <svg class="w-8 h-8 text-gray-400 mb-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6"></path>
              </svg>
              <span class="text-sm text-gray-500">{{ getText('uploadDocuments') || 'Загрузить документы' }}</span>
            </div>
          </div>
          <div v-if="formData.documentPhoto" class="mt-2 text-sm text-green-600">
            ✓ {{ formData.documentPhoto.name }}
          </div>
          <div v-if="fileErrors.documentPhoto" class="mt-2 text-sm text-red-600">
            {{ fileErrors.documentPhoto }}
          </div>
        </div>

        <div class="flex gap-3 w-full max-w-xs">
          <button @click="prevStep" 
            class="bg-gray-200 hover:bg-gray-300 text-gray-700 font-semibold py-3 px-6 rounded-full flex-1 transition-colors duration-200">
            {{ getText('backButton') || 'Назад' }}
          </button>
          <button @click="nextStep" :disabled="!canProceed()"
            class="bg-[#FF2D6A] hover:bg-[#e0265c] text-white font-semibold py-3 px-6 rounded-full flex-1 transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed">
            {{ getText('continueButton') }}
          </button>
        </div>
      </div>

      <!-- Step 8: Contact Information (7/10) -->
      <div v-else-if="currentStep === 8" class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-2 text-center">{{ getText('contactTitle') || 'КОНТАКТНАЯ ИНФОРМАЦИЯ' }} 7/10</h3>
        <p class="text-sm text-gray-600 mb-6 text-center">{{ getText('contactSubtitle') || 'Укажите ваши контактные данные' }}</p>
        
        <div class="w-full space-y-4 mb-6">
          <div>
            <input v-model="formData.mobilePhone" 
              :placeholder="getText('mobilePhonePlaceholder') || 'Мобильный телефон *'"
              class="w-full rounded-xl border border-gray-200 px-4 py-3 focus:outline-none focus:ring-2 focus:ring-[#FF2D6A]">
          </div>
          <div>
            <input v-model="formData.officePhone" 
              :placeholder="getText('officePhonePlaceholder') || 'Рабочий телефон (необязательно)'"
              class="w-full rounded-xl border border-gray-200 px-4 py-3 focus:outline-none focus:ring-2 focus:ring-[#FF2D6A]">
          </div>
          <div>
            <input v-model="formData.email" type="email"
              :placeholder="getText('emailPlaceholder') || 'Email *'"
              class="w-full rounded-xl border border-gray-200 px-4 py-3 focus:outline-none focus:ring-2 focus:ring-[#FF2D6A]">
          </div>
          <label class="flex items-center space-x-3 cursor-pointer">
            <input type="checkbox" v-model="formData.allowCallback"
              class="w-5 h-5 text-[#FF2D6A] border-gray-300 rounded focus:ring-[#FF2D6A]">
            <span class="text-sm text-gray-700">{{ getText('allowCallbackLabel') || 'Разрешаю обратный звонок для уточнения деталей' }}</span>
          </label>
        </div>

        <div class="flex gap-3 w-full max-w-xs">
          <button @click="prevStep" 
            class="bg-gray-200 hover:bg-gray-300 text-gray-700 font-semibold py-3 px-6 rounded-full flex-1 transition-colors duration-200">
            {{ getText('backButton') || 'Назад' }}
          </button>
          <button @click="nextStep" :disabled="!canProceed()"
            class="bg-[#FF2D6A] hover:bg-[#e0265c] text-white font-semibold py-3 px-6 rounded-full flex-1 transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed">
            {{ getText('continueButton') }}
          </button>
        </div>
      </div>

      <!-- Step 9: Selfie Video (8/10) -->
      <div v-else-if="currentStep === 9" class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-2 text-center">{{ getText('selfieVideoTitle') || 'СЕЛФИ-ВИДЕО' }} 8/10</h3>
        <p class="text-sm text-gray-600 mb-6 text-center">{{ getText('selfieVideoSubtitle') || 'Запишите короткое видео с собой для подтверждения' }}</p>
        
        <div class="w-full mb-6">
          <div class="relative">
            <input type="file" @change="handleFileUpload($event, 'selfieVideo')" accept="video/*"
              class="absolute inset-0 w-full h-full opacity-0 cursor-pointer">
            <div class="w-full h-40 bg-gray-50 border-2 border-dashed border-gray-300 rounded-xl flex flex-col items-center justify-center hover:bg-gray-100 transition-colors">
              <svg class="w-12 h-12 text-gray-400 mb-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 10l4.553-2.276A1 1 0 0121 8.618v6.764a1 1 0 01-1.447.894L15 14M5 18h8a2 2 0 002-2V8a2 2 0 00-2-2H5a2 2 0 00-2 2v8a2 2 0 002 2z"></path>
              </svg>
              <span class="text-sm text-gray-500 text-center">{{ getText('uploadSelfieVideo') || 'Загрузить селфи-видео' }}</span>
              <span class="text-xs text-gray-400 mt-1">{{ getText('videoRequirements') || 'Макс. 50 МБ, до 2 минут' }}</span>
            </div>
          </div>
          <div v-if="formData.selfieVideo" class="mt-2 text-sm text-green-600">
            ✓ {{ formData.selfieVideo.name }}
          </div>
          <div v-if="fileErrors.selfieVideo" class="mt-2 text-sm text-red-600">
            {{ fileErrors.selfieVideo }}
          </div>
        </div>

        <div class="flex gap-3 w-full max-w-xs">
          <button @click="prevStep" 
            class="bg-gray-200 hover:bg-gray-300 text-gray-700 font-semibold py-3 px-6 rounded-full flex-1 transition-colors duration-200">
            {{ getText('backButton') || 'Назад' }}
          </button>
          <button @click="nextStep" :disabled="!canProceed()"
            class="bg-[#FF2D6A] hover:bg-[#e0265c] text-white font-semibold py-3 px-6 rounded-full flex-1 transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed">
            {{ getText('continueButton') }}
          </button>
        </div>
      </div>

      <!-- Step 10: Summary/Confirmation (10/10) -->
      <div v-else-if="currentStep === 10" class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-6 text-center">{{ getText('confirmationTitle') || 'ПОДТВЕРЖДЕНИЕ ДАННЫХ' }} 10/10</h3>
        <div class="w-full mb-6 max-h-64 overflow-y-auto text-sm text-gray-700 space-y-2">
          <div><strong>{{ getText('nameTitle') }}:</strong> {{ formData.name }}</div>
          <div><strong>{{ getText('promotionTitle') }}:</strong> {{ formData.promotionDescription }}</div>
          <div><strong>{{ getText('geographyTitle') }}:</strong> {{ formData.geography }}</div>
          <div><strong>{{ getText('languagesTitle') || 'Языки' }}:</strong> {{ formData.promotionLanguages.join(', ') }}</div>
          <div><strong>{{ getText('locationTitle') || 'Местоположение' }}:</strong> {{ formData.manualAddress || formData.location }}</div>
          <div><strong>{{ getText('contactTitle') || 'Контакты' }}:</strong> {{ formData.mobilePhone }}, {{ formData.email }}</div>
          <div v-if="formData.businessPhoto"><strong>{{ getText('businessPhotoLabel') || 'Фото бизнеса' }}:</strong> ✓</div>
          <div v-if="formData.documentPhoto"><strong>{{ getText('documentPhotoLabel') || 'Документы' }}:</strong> ✓</div>
          <div v-if="formData.selfieVideo"><strong>{{ getText('selfieVideoTitle') || 'Селфи-видео' }}:</strong> ✓</div>
        </div>
        <div class="flex gap-3 w-full max-w-xs">
          <button @click="prevStep" 
            class="bg-gray-200 hover:bg-gray-300 text-gray-700 font-semibold py-3 px-6 rounded-full flex-1 transition-colors duration-200">
            {{ getText('backButton') || 'Назад' }}
          </button>
          <button @click="submitForm"
            class="bg-[#FF2D6A] hover:bg-[#e0265c] text-white font-semibold py-3 px-6 rounded-full flex-1 transition-colors duration-200">
            {{ getText('submitButton') || 'ОТПРАВИТЬ' }}
          </button>
        </div>
      </div>
      <!-- Completion Screen -->
      <div v-else class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-2 text-center">{{ getText('thankYouTitle') || 'Спасибо!' }}</h3>
        <p class="text-gray-600 text-center mb-8">{{ getText('thankYouMessage') || 'Ваша заявка отправлена на рассмотрение.' }}</p>
        <button @click="closeModal"
          class="w-full max-w-xs bg-[#FF2D6A] text-white font-semibold py-3 rounded-full transition-colors duration-200">
          {{ getText('closeButton') || 'Закрыть' }}
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
      selectedLanguage: '',
      currentLanguage: 'ru',
      formData: {
        name: '',
        isRepresentative: false,
        confirmAccuracy: false,
        promotionDescription: '',
        geography: '',
        promotionLanguages: [],
        location: '',
        manualAddress: '',
        businessPhoto: null,
        documentPhoto: null,
        allowCallback: false,
        mobilePhone: '',
        officePhone: '',
        email: '',
        selfieVideo: null
      },
      fileErrors: {},
      fileSizes: {},
      texts: {
        uz: {
          language: 'TILNI TANLANG',
          selectLanguage: 'Tilni tanlang',
          termsTitle: 'DAVOM ETISHDAN OLDIN O\'QING',
          responsibilityTitle: 'Mas\'uliyat:',
          responsibilityText: 'Iltimos, faqat ishonchli va dolzarb ma\'lumotlarni kiriting. Quyidagi ma\'lumotlarni yuborish taqiqlanadi:',
          responsibility1: 'boshqa shaxslar nomidan ularning xabari yo\'q holda;',
          responsibility2: 'soxta takliflar;',
          responsibility3: 'chalg\'ituvchi formulalar.',
          publicityTitle: 'Oshkoralik:',
          publicityText: 'Agar ma\'lumotlaringiz tekshiruvdan o\'tsa, ular quyidagi joylarda joylashtirilishi mumkin:',
          publicity1: 'Radio Yonar efirida;',
          publicity2: 'bizning ijtimoiy tarmoqlarimizda;',
          publicity3: 'rasmiy platformalarda.',
          consentTitle: 'Rozilik:',
          consentText: '"Roziman" tugmasini bosish orqali siz ushbu shartlarga roziligingizni tasdiqlaysiz.',
          agreeButton: 'MEN ROZIMAN',
          nameTitle: 'ISMINGIZNI KIRITING:',
          namePlaceholder: 'Ismingizni kiriting',
          representativeCheck: 'Men kompaniyaning rasmiy vakili yoki egasiman.',
          accuracyCheck: 'Men taqdim etayotgan ma\'lumotlarning to\'g\'riligini tasdiqlayman.',
          continueButton: 'DAVOM ETISH',
          promotionTitle: 'AKSIYANGIZNI TASVIRLAB BERING',
          examplesTitle: 'Format namunalari:',
          example1: '10 avgustgacha — 2+1 bepul',
          example2: '50% gacha chegirmalar',
          example3: '400 000 so\'mdan xarid qilganda sovg\'a',
          example4: 'Har 10-avtomobil yuvish bepul',
          example5: 'Yozgi kolleksiyaning yakuniy sotuvi',
          promotionPlaceholder: 'Aksiya turini tasvirlab bering. Muhim! (Batafsil tasvirlab bering: amal qilish muddati va aksiya shartlari, yosh cheklovi va hokazo.)',
          geographyTitle: 'AKSIYA GEOGRAFIYASI',
          geographySubtitle: 'Aksiya qaysi geografiyaga mo\'ljallangan? Qo\'lda mintaqani kiriting: Mamlakat, shahar, viloyat.',
          geographyPlaceholder: 'Mamlakat, shahar, viloyat'
        },
        ru: {
          language: 'ВЫБЕРИТЕ ЯЗЫК',
          selectLanguage: 'Выберите язык',
          termsTitle: 'ОЗНАКОМЬТЕСЬ ПЕРЕД ПРОДОЛЖЕНИЕМ',
          responsibilityTitle: 'Ответственность:',
          responsibilityText: 'Пожалуйста, вводите только достоверную и актуальную информацию. Запрещается отправлять данные:',
          responsibility1: 'от имени других лиц без их ведома;',
          responsibility2: 'фейковые предложения;',
          responsibility3: 'вводящие в заблуждение формулировки.',
          publicityTitle: 'Публичность:',
          publicityText: 'Если ваша информация пройдет верификацию, она может быть размещена:',
          publicity1: 'в эфире Radio Yonar;',
          publicity2: 'в наших соцсетях;',
          publicity3: 'на официальных платформах.',
          consentTitle: 'Согласие:',
          consentText: 'Нажимая кнопку "Согласен", вы подтверждаете, что согласны с этими условиями.',
          agreeButton: 'Я СОГЛАСЕН',
          nameTitle: 'УКАЖИТЕ СВОЁ ИМЯ:',
          namePlaceholder: 'Введите ваше имя',
          representativeCheck: 'Я являюсь официальным представителем или владельцем компании.',
          accuracyCheck: 'Я подтверждаю достоверность предоставляемой мной информации.',
          continueButton: 'ПРОДОЛЖИТЬ',
          promotionTitle: 'ОПИСАНИЕ ВАШЕЙ АКЦИИ',
          examplesTitle: 'Примеры форматов:',
          example1: 'До 10 августа — 2+1 бесплатно',
          example2: 'Скидки до 50%',
          example3: 'Подарок при покупке от 400 000 сум',
          example4: 'Каждая 10-я мойка авто бесплатно',
          example5: 'Финальная распродажа летней коллекции',
          promotionPlaceholder: 'Опишите тип своей акции Важно! (Опишите подробно: срок действия и условия акции, возрастное ограничение и т.д.)',
          geographyTitle: 'ГЕОГРАФИЯ АКЦИИ',
          geographySubtitle: 'На какую географию рассчитана акция? Введите вручную регион: Страна, город, провинция.',
          geographyPlaceholder: 'Страна, город, провинция'
        },
        en: {
          language: 'SELECT LANGUAGE',
          selectLanguage: 'Select language',
          termsTitle: 'READ BEFORE CONTINUING',
          responsibilityTitle: 'Responsibility:',
          responsibilityText: 'Please enter only reliable and up-to-date information. It is prohibited to send data:',
          responsibility1: 'on behalf of other persons without their knowledge;',
          responsibility2: 'fake offers;',
          responsibility3: 'misleading formulations.',
          publicityTitle: 'Publicity:',
          publicityText: 'If your information passes verification, it may be posted:',
          publicity1: 'on Radio Yonar broadcast;',
          publicity2: 'on our social networks;',
          publicity3: 'on official platforms.',
          consentTitle: 'Consent:',
          consentText: 'By clicking the "Agree" button, you confirm that you agree to these terms.',
          agreeButton: 'I AGREE',
          nameTitle: 'ENTER YOUR NAME:',
          namePlaceholder: 'Enter your name',
          representativeCheck: 'I am an official representative or owner of the company.',
          accuracyCheck: 'I confirm the accuracy of the information I provide.',
          continueButton: 'CONTINUE',
          promotionTitle: 'DESCRIPTION OF YOUR PROMOTION',
          examplesTitle: 'Format examples:',
          example1: 'Until August 10 — 2+1 free',
          example2: 'Discounts up to 50%',
          example3: 'Gift with purchase from 400,000 sum',
          example4: 'Every 10th car wash free',
          example5: 'Final sale of summer collection',
          promotionPlaceholder: 'Describe your promotion type Important! (Describe in detail: validity period and promotion conditions, age restrictions, etc.)',
          geographyTitle: 'PROMOTION GEOGRAPHY',
          geographySubtitle: 'What geography is the promotion designed for? Enter region manually: Country, city, province.',
          geographyPlaceholder: 'Country, city, province'
        }
      }
    }
  },
  methods: {
    getText(key) {
      return this.texts[this.currentLanguage][key]
    },
    getTitle(step) {
      const titles = {
        language: this.getText('language'),
        terms: this.getText('termsTitle'),
        name: this.getText('nameTitle'),
        promotion: this.getText('promotionTitle'),
        geography: this.getText('geographyTitle')
      }
      return titles[step] || ''
    },
    getSubtitle(step) {
      const subtitles = {
        geography: this.getText('geographySubtitle')
      }
      return subtitles[step] || ''
    },
    getPlaceholder(key) {
      const placeholders = {
        selectLanguage: this.getText('selectLanguage'),
        name: this.getText('namePlaceholder'),
        promotion: this.getText('promotionPlaceholder'),
        geography: this.getText('geographyPlaceholder')
      }
      return placeholders[key] || ''
    },
    setLanguageAndContinue() {
      if (this.selectedLanguage) {
        this.currentLanguage = this.selectedLanguage
        this.currentStep = 1
      }
    },
    agreeToTerms() {
      this.currentStep = 2
    },
    nextStep() {
      if (!this.canProceed()) {
        return
      }

      if (this.currentStep < 9) {
        this.currentStep++
      } else {
        this.submitForm()
      }
    },
    prevStep() {
      if (this.currentStep > 0) {
        this.currentStep--
      }
    },
    submitForm() {
      this.$emit('submit', this.formData)
    },
    handleFileUpload(event, fieldName) {
      const file = event.target.files[0]
      if (file) {
        const maxSize = 50 * 1024 * 1024
        if (file.size > maxSize) {
          this.$set(this.fileErrors, fieldName, this.getText('fileSizeError') || 'File size must not exceed 50 MB')
          this.$set(this.fileSizes, fieldName, file.size)
          return
        } else {
          this.$set(this.fileErrors, fieldName, null)
        }

        this.formData[fieldName] = file
        this.$set(this.fileSizes, fieldName, file.size)
      }
    },
    formatFileSize(bytes) {
      if (bytes === 0) return '0 Bytes'
      const k = 1024
      const sizes = ['Bytes', 'KB', 'MB', 'GB']
      const i = Math.floor(Math.log(bytes) / Math.log(k))
      return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i]
    },
    isValidName(name) {
      if (!name || typeof name !== 'string') return false
      const letterCount = (name.match(/[a-zA-Z\u0410-\u042f\u0430-\u044f\u0401\u0451\u045e\u049b\u0493\u04b3]/g) || []).length
      const hasOnlyLettersAndSpaces = /^[a-zA-Z\u0410-\u042f\u0430-\u044f\u0401\u0451\u045e\u049b\u0493\u04b3\s]+$/.test(name.trim())
      return letterCount >= 3 && hasOnlyLettersAndSpaces
    },
    canProceed() {
      if (!this.formData) return false

      switch (this.currentStep) {
        case 0:
          return !!this.selectedLanguage
        case 1:
          return true
        case 2:
          return this.isValidName(this.formData.name || '') &&
            this.formData.isRepresentative &&
            this.formData.confirmAccuracy
        case 3:
          return (this.formData.promotionDescription || '').trim().length > 0
        case 4:
          return (this.formData.geography || '').trim().length > 0
        case 5:
          return (this.formData.promotionLanguages || []).length > 0
        case 6:
          return (this.formData.location || '').trim().length > 0 || (this.formData.manualAddress || '').trim().length > 0
        case 7:
          return this.formData.businessPhoto || this.formData.documentPhoto
        case 8:
          return (this.formData.mobilePhone || '').trim().length > 0 && (this.formData.email || '').trim().length > 0
        case 9:
          return !!this.formData.selfieVideo
        default:
          return true
      }
    },

    handleFileUpload(event, fieldName) {
      const file = event.target.files[0]
      if (!file) return

      // Check file size (50MB limit)
      const maxSize = 50 * 1024 * 1024 // 50MB in bytes
      if (file.size > maxSize) {
        this.fileErrors[fieldName] = 'File size must be 50MB or less'
        this.formData[fieldName] = null
        return
      }

      // Clear any previous errors
      this.fileErrors[fieldName] = ''
      
      // Set the file directly (Vue 3 reactivity)
      this.formData[fieldName] = file
      this.fileSizes[fieldName] = file.size
    },

    submitForm() {
      // Emit the form data to parent component
      this.$emit('submit', this.formData)
      this.currentStep = 11 // Move to completion screen
    },

    closeModal() {
      this.currentStep = 0
      this.selectedLanguage = ''
      this.currentLanguage = 'ru'
      this.formData = {
        name: '',
        isRepresentative: false,
        confirmAccuracy: false,
        promotionDescription: '',
        geography: '',
        promotionLanguages: [],
        location: '',
        manualAddress: '',
        businessPhoto: null,
        documentPhoto: null,
        allowCallback: false,
        mobilePhone: '',
        officePhone: '',
        email: '',
        selfieVideo: null
      }
      this.fileErrors = {}
      this.fileSizes = {}
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