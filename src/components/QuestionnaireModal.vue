<template>
  <div class="flex flex-col items-center justify-center w-full min-h-screen p-4">
    <!-- Main Container -->
    <div class="w-full max-w-md">
      <!-- Progress Circle - 50% visible, behind the cards, peeking from top -->
      <div v-if="currentStep > 0 && currentStep <= 11" class="relative flex justify-center mb-6" style="height: 0;">
        <div 
          class="w-24 h-24 bg-white rounded-full shadow-md border-2 border-gray-200 flex items-center justify-center"
          style="
            position: absolute;
            top: -18px;
            left: 50%;
            transform: translateX(-50%);
            z-index: -1;
            pointer-events: none;
          "
        ></div>
      </div>
      
      <!-- Show normal questionnaire content for steps 0-11 -->
      <div v-if="currentStep <= 11" class="w-full bg-white rounded-3xl shadow-md p-6">
      <!-- Language Selection -->
      <div v-if="currentStep === 0" class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-2 text-center">{{ getTitle('language') }}</h3>
        <p class="text-sm text-gray-600 mb-6 text-center">{{ getSubtitle('language') }}</p>

        <div class="w-full mb-6">
          <select v-model="selectedLanguage"
            class="w-full rounded-xl border border-gray-200 px-4 py-3 focus:outline-none focus:ring-2 focus:ring-green-500 text-center">
            <option value="">{{ getPlaceholder('selectLanguage') }}</option>
            <option value="uz">O'ZBEKCHA</option>
            <option value="ru">РУССКИЙ</option>
            <option value="en">ENGLISH</option>
          </select>
        </div>

        <div class="flex justify-center">
          <button @click="setLanguageAndContinue" :disabled="!selectedLanguage"
            class="bg-[#c0d700] hover:bg-[#a8c000] text-white font-semibold py-3 px-12 rounded-lg transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed">
            OK
          </button>
        </div>
      </div>

      <!-- Terms & Conditions -->
      <div v-else-if="currentStep === 1" class="w-full">
        <h3 class="text-xl font-bold text-gray-800 mb-6 text-center">{{ getText('termsTitle') || 'ОЗНАКОМЬТЕСЬ ПЕРЕД ПРОДОЛЖЕНИЕМ' }}</h3>

        <div class="w-full mb-8 space-y-6">
          <!-- Responsibility Section -->
          <div>
            <h4 class="font-bold text-gray-800 mb-3">{{ getText('responsibilityTitle') || 'Ответственность:' }}</h4>
            <p class="text-gray-700 mb-3 leading-relaxed">{{ getText('responsibilityText') || 'Пожалуйста, вводите только достоверную и актуальную информацию.' }}</p>
            <ul class="text-gray-700 space-y-2 ml-4">
              <li class="flex items-start">
                <span class="w-2 h-2 bg-gray-800 rounded-full mt-2 mr-3 flex-shrink-0"></span>
                <span>{{ getText('responsibility1') || 'от имени других лиц без их ведома;' }}</span>
              </li>
              <li class="flex items-start">
                <span class="w-2 h-2 bg-gray-800 rounded-full mt-2 mr-3 flex-shrink-0"></span>
                <span>{{ getText('responsibility2') || 'фейковые предложения;' }}</span>
              </li>
              <li class="flex items-start">
                <span class="w-2 h-2 bg-gray-800 rounded-full mt-2 mr-3 flex-shrink-0"></span>
                <span>{{ getText('responsibility3') || 'вводящие в заблуждение формулировки.' }}</span>
              </li>
            </ul>
          </div>

          <!-- Publicity Section -->
          <div>
            <h4 class="font-bold text-gray-800 mb-3">{{ getText('publicityTitle') || 'Публичность:' }}</h4>
            <p class="text-gray-700 mb-3 leading-relaxed">{{ getText('publicityText') || 'Если ваша информация пройдет верификацию, она может быть размещена:' }}</p>
            <ul class="text-gray-700 space-y-2 ml-4">
              <li class="flex items-start">
                <span class="w-2 h-2 bg-gray-800 rounded-full mt-2 mr-3 flex-shrink-0"></span>
                <span>{{ getText('publicity1') || 'в эфире Radio Yonar;' }}</span>
              </li>
              <li class="flex items-start">
                <span class="w-2 h-2 bg-gray-800 rounded-full mt-2 mr-3 flex-shrink-0"></span>
                <span>{{ getText('publicity2') || 'в наших соцсетях;' }}</span>
              </li>
              <li class="flex items-start">
                <span class="w-2 h-2 bg-gray-800 rounded-full mt-2 mr-3 flex-shrink-0"></span>
                <span>{{ getText('publicity3') || 'на официальных платформах.' }}</span>
              </li>
            </ul>
          </div>

          <!-- Consent Section -->
          <div>
            <h4 class="font-bold text-gray-800 mb-3">{{ getText('consentTitle') || 'Согласие:' }}</h4>
            <p class="text-gray-700 leading-relaxed">{{ getText('consentText') || 'Нажимая кнопку "Согласен", вы подтверждаете, что согласны с этими условиями.' }}</p>
          </div>
        </div>

        <div class="flex justify-center">
          <button @click="nextStep"
            class="bg-[#c0d700] hover:bg-[#a8c000] text-white font-semibold py-3 px-12 rounded-lg transition-colors duration-200">
            {{ getText('agreeButton') || 'Согласен' }}
          </button>
        </div>
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

        <div class="flex justify-center">
          <button @click="nextStep" :disabled="!canProceed()"
            class="bg-[#c0d700] hover:bg-[#a8c000] text-white font-semibold py-3 px-12 rounded-lg transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed">
            {{ getText('continueButton') }}
          </button>
        </div>
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

        <div class="flex justify-center">
          <button @click="nextStep" :disabled="!formData.promotionDescription?.trim()"
            class="bg-[#c0d700] hover:bg-[#a8c000] text-white font-semibold py-3 px-12 rounded-lg transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed">
            {{ getText('continueButton') }}
          </button>
        </div>
      </div>

      <!-- Step 3: Geography (3/10) -->
      <div v-else-if="currentStep === 4" class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-2 text-center">{{ getText('geographyTitle') }} 3/10</h3>
        <p class="text-sm text-gray-600 mb-6 text-center">{{ getText('geographySubtitle') }}</p>

        <input v-model="formData.geography" :placeholder="getText('geographyPlaceholder')"
          class="w-full rounded-xl border border-gray-200 px-4 py-3 mb-6 focus:outline-none focus:ring-2 focus:ring-[#FF2D6A]" />

        <div class="flex justify-center">
          <button @click="nextStep" :disabled="!formData.geography?.trim()"
            class="bg-[#c0d700] hover:bg-[#a8c000] text-white font-semibold py-3 px-12 rounded-lg transition-colors duration-200 disabled:opacity-50 disabled:cursor-not-allowed">
            {{ getText('continueButton') }}
          </button>
        </div>
      </div>

      <!-- Step 5: Promotion Languages (4/10) -->
      <div v-else-if="currentStep === 5" class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-2 text-center">{{ getText('languagesTitle') || 'ЯЗЫКИ АКЦИИ' }} 4/10</h3>
        <p class="text-sm text-gray-600 mb-6 text-center">{{ getText('languagesSubtitle') || 'На каких языках будет проводиться акция?' }}</p>
        <div class="w-full mb-6 space-y-3">
          <label v-for="lang in ['Узбекский', 'Русский', 'Английский']" :key="lang" 
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

      <!-- Step 7: Business Photo/Video Upload (6/10) -->
      <div v-else-if="currentStep === 7" class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-2 text-center">{{ getText('businessPhotoTitle') || 'ФОТО ИЛИ ВИДЕО (ОБЯЗАТЕЛЬНО)' }} 6/10</h3>
        <p class="text-sm text-gray-600 mb-6 text-center leading-relaxed">
          {{ getText('businessPhotoSubtitle') || 'Фото вашей точки / магазина / офиса или товара / процесса оказания услуги' }}<br>
          <span class="text-xs text-gray-500">{{ getText('businessPhotoNote') || '(вы должны присутствовать в кадре)' }}</span>
        </p>
        
        <div class="w-full mb-6">
          <div class="relative">
            <input type="file" @change="handleFileUpload($event, 'businessPhoto')" accept="image/*,video/*"
              class="absolute inset-0 w-full h-full opacity-0 cursor-pointer">
            <div class="w-full h-40 bg-gray-50 border-2 border-dashed border-gray-300 rounded-xl flex flex-col items-center justify-center hover:bg-gray-100 transition-colors">
              <svg class="w-12 h-12 text-gray-400 mb-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4"></path>
              </svg>
              <span class="text-sm font-medium text-gray-600">{{ getText('uploadButton') || 'ЗАГРУЗИТЬ' }}</span>
              <span class="text-xs text-gray-400 mt-1">{{ getText('uploadHint') || 'Камера или галерея' }}</span>
            </div>
          </div>
          <div v-if="formData.businessPhoto" class="mt-3 p-3 bg-green-50 rounded-lg">
            <div class="flex items-center text-sm text-green-700">
              <svg class="w-4 h-4 mr-2" fill="currentColor" viewBox="0 0 20 20">
                <path fill-rule="evenodd" d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z" clip-rule="evenodd"></path>
              </svg>
              {{ getText('imageUploaded') || 'Изображение загружено' }}: {{ formData.businessPhoto.name }}
            </div>
          </div>
          <div v-if="fileErrors.businessPhoto" class="mt-2 text-sm text-red-600">
            {{ fileErrors.businessPhoto }}
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

      <!-- Step 8: Document Photo/Video Upload (7/10) -->
      <div v-else-if="currentStep === 8" class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-2 text-center">{{ getText('documentPhotoTitle') || 'ФОТО ИЛИ ВИДЕО (ОБЯЗАТЕЛЬНО)' }} 7/10</h3>
        <p class="text-sm text-gray-600 mb-6 text-center leading-relaxed">
          {{ getText('documentPhotoSubtitle') || 'Фото или видео лицензии, сертификата или любого официального документа' }}<br>
          <span class="text-xs text-gray-500">{{ getText('documentPhotoNote') || '(сильно поможет при верификации)' }}</span>
        </p>
        
        <div class="w-full mb-6">
          <div class="relative">
            <input type="file" @change="handleFileUpload($event, 'documentPhoto')" accept="image/*,video/*"
              class="absolute inset-0 w-full h-full opacity-0 cursor-pointer">
            <div class="w-full h-40 bg-gray-50 border-2 border-dashed border-gray-300 rounded-xl flex flex-col items-center justify-center hover:bg-gray-100 transition-colors">
              <svg class="w-12 h-12 text-gray-400 mb-3" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4"></path>
              </svg>
              <span class="text-sm font-medium text-gray-600">{{ getText('uploadButton') || 'ЗАГРУЗИТЬ' }}</span>
              <span class="text-xs text-gray-400 mt-1">{{ getText('uploadHint') || 'Камера или галерея' }}</span>
            </div>
          </div>
          <div v-if="formData.documentPhoto" class="mt-3 p-3 bg-green-50 rounded-lg">
            <div class="flex items-center text-sm text-green-700">
              <svg class="w-4 h-4 mr-2" fill="currentColor" viewBox="0 0 20 20">
                <path fill-rule="evenodd" d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z" clip-rule="evenodd"></path>
              </svg>
              {{ getText('imageUploaded') || 'Изображение загружено' }}: {{ formData.documentPhoto.name }}
            </div>
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

      <!-- Step 9: Contact Information (8/10) -->
      <div v-else-if="currentStep === 9" class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-2 text-center">{{ getText('contactTitle') || 'КОНТАКТНАЯ ИНФОРМАЦИЯ' }} 8/10</h3>
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

      <!-- Step 10: Selfie Video (9/10) -->
      <div v-else-if="currentStep === 10" class="w-full flex flex-col items-center">
        <h3 class="text-lg font-semibold text-gray-800 mb-2 text-center">{{ getText('selfieVideoTitle') || 'СЕЛФИ-ВИДЕО' }} 9/10</h3>
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

      <!-- Step 11: Initial Completion Screen (10/10) -->
      <div v-else-if="currentStep === 11" class="w-full flex flex-col items-center">
        <div class="text-center mb-6">
          <h3 class="text-xl font-bold text-gray-800 mb-2">{{ getText('congratulationsTitle') || 'Поздравляем, все этапы завершены!' }}</h3>
          <div class="text-lg font-semibold text-[#FF2D6A] mb-4">10/10</div>
          <p class="text-sm text-gray-600 mb-6">{{ getText('reviewMessage') || 'Пожалуйста, проверьте все ваши ответы перед отправкой' }}</p>
          
          <!-- Review status message -->
          <div v-if="!hasReviewed" class="w-full mb-4 p-3 bg-blue-50 border border-blue-200 rounded-lg">
            <p class="text-sm text-blue-700">
              <strong>{{ getText('reviewRecommended') || 'Рекомендуется:' }}</strong> 
              {{ getText('previewRecommended') || 'Нажмите "ПРЕДВАРИТЕЛЬНЫЙ ПРОСМОТР" чтобы проверить ваши данные перед отправкой' }}
            </p>
          </div>
          
          <div v-else class="w-full mb-4 p-3 bg-green-50 border border-green-200 rounded-lg">
            <div class="flex items-center text-sm text-green-700">
              <svg class="w-4 h-4 mr-2" fill="currentColor" viewBox="0 0 20 20">
                <path fill-rule="evenodd" d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z" clip-rule="evenodd"></path>
              </svg>
              {{ getText('reviewCompleted') || 'Просмотр завершен! Данные проверены' }}
            </div>
          </div>
        </div>
        
        <!-- Preview Button -->
        <button @click="showPreview = true"
          class="w-full max-w-xs bg-blue-500 hover:bg-blue-600 text-white font-semibold py-3 px-6 rounded-lg transition-colors duration-200 mb-3">
          {{ getText('previewButton') || 'ПРЕДВАРИТЕЛЬНЫЙ ПРОСМОТР' }}
        </button>
        
        <!-- Continue Button -->
        <button @click="nextStep"
          class="w-full max-w-xs bg-[#c0d700] hover:bg-[#a8c000] text-white font-semibold py-3 px-6 rounded-lg transition-colors duration-200 mb-3">
          {{ getText('continueButton') || 'ПРОДОЛЖИТЬ' }}
        </button>
        <div class="text-xs text-gray-500 mt-1">Debug: Step {{ currentStep }}</div>
        
        <!-- Back Button -->
        <button @click="prevStep"
          class="w-full max-w-xs bg-gray-200 hover:bg-gray-300 text-gray-700 font-semibold py-3 px-6 rounded-lg transition-colors duration-200">
          {{ getText('backButton') || 'Назад' }}
        </button>
      </div>
      
      <!-- Preview Modal Overlay -->
      <div v-if="showPreview" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50" @click="showPreview = false">
        <div class="bg-white rounded-2xl shadow-xl w-full max-w-md mx-4 max-h-[80vh] overflow-hidden" @click.stop>
          <div class="p-6">
            <div class="flex justify-between items-center mb-4">
              <h3 class="text-lg font-semibold text-gray-800">{{ getText('previewTitle') || 'Предварительный просмотр данных' }}</h3>
              <button @click="showPreview = false" class="text-gray-400 hover:text-gray-600">
                <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
                </svg>
              </button>
            </div>
            
            <div class="mb-4 p-3" :class="hasReviewed ? 'bg-green-50 border border-green-200' : 'bg-blue-50 border border-blue-200'">
              <div class="flex items-center justify-between">
                <div>
                  <p class="text-sm" :class="hasReviewed ? 'text-green-700' : 'text-blue-700'">
                    <strong>{{ hasReviewed ? (getText('reviewCompleted') || 'Просмотр завершен!') : (getText('reviewNotice') || 'Внимательно проверьте все данные:') }}</strong>
                  </p>
                  <ul class="text-xs mt-1 list-disc list-inside space-y-1" :class="hasReviewed ? 'text-green-600' : 'text-blue-600'">
                    <li>{{ getText('reviewName') || 'Правильность имени' }}</li>
                    <li>{{ getText('reviewContact') || 'Контактные данные' }}</li>
                    <li>{{ getText('reviewFiles') || 'Загруженные файлы' }}</li>
                    <li>{{ getText('reviewPromotion') || 'Информация об акции' }}</li>
                  </ul>
                </div>
                <div v-if="hasReviewed" class="text-green-500">
                  <svg class="w-6 h-6" fill="currentColor" viewBox="0 0 20 20">
                    <path fill-rule="evenodd" d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z" clip-rule="evenodd"></path>
                  </svg>
                </div>
              </div>
            </div>
            
            <div class="max-h-96 overflow-y-auto text-sm text-gray-700 space-y-3">
              <div><strong>{{ getText('nameTitle') || 'Имя' }}:</strong> {{ formData.name }}</div>
              <div><strong>{{ getText('representativeLabel') || 'Представитель' }}:</strong> {{ formData.isRepresentative ? 'Да' : 'Нет' }}</div>
              <div><strong>{{ getText('accuracyLabel') || 'Точность данных' }}:</strong> {{ formData.confirmAccuracy ? 'Подтверждено' : 'Не подтверждено' }}</div>
              <div><strong>{{ getText('promotionTitle') || 'Описание акции' }}:</strong> {{ formData.promotionDescription }}</div>
              <div><strong>{{ getText('geographyTitle') || 'География' }}:</strong> {{ formData.geography }}</div>
              <div v-if="formData.promotionLanguages && formData.promotionLanguages.length > 0">
                <strong>{{ getText('languagesTitle') || 'Языки акции' }}:</strong> {{ formData.promotionLanguages.join(', ') }}
              </div>
              <div v-if="formData.manualAddress || formData.location">
                <strong>{{ getText('locationTitle') || 'Местоположение' }}:</strong> {{ formData.manualAddress || formData.location }}
              </div>
              <div><strong>{{ getText('contactTitle') || 'Контакты' }}:</strong></div>
              <div class="ml-4">
                <div>Мобильный: {{ formData.mobilePhone }}</div>
                <div v-if="formData.officePhone">Рабочий: {{ formData.officePhone }}</div>
                <div>Email: {{ formData.email }}</div>
                <div>Обратный звонок: {{ formData.allowCallback ? 'Разрешен' : 'Запрещен' }}</div>
              </div>
              <div><strong>{{ getText('uploadsTitle') || 'Загруженные файлы' }}:</strong></div>
              <div class="ml-4">
                <div v-if="formData.businessPhoto">✓ {{ getText('businessPhotoLabel') || 'Фото бизнеса' }}: {{ formData.businessPhoto.name }}</div>
                <div v-if="formData.documentPhoto">✓ {{ getText('documentPhotoLabel') || 'Документы' }}: {{ formData.documentPhoto.name }}</div>
                <div v-if="formData.selfieVideo">✓ {{ getText('selfieVideoTitle') || 'Селфи-видео' }}: {{ formData.selfieVideo.name }}</div>
              </div>
            </div>
            
            <!-- 4 Action Buttons in Preview Modal -->
            <div class="mt-6 space-y-3">
              <button @click="editForm"
                class="w-full bg-orange-500 hover:bg-orange-600 text-white font-semibold py-3 px-4 rounded-full transition-colors duration-200">
                {{ getText('editButton') || 'РЕДАКТИРОВАТЬ' }}
              </button>
              <button @click="markAsReviewed"
                class="w-full bg-orange-500 hover:bg-orange-600 text-white font-semibold py-3 px-4 rounded-full transition-colors duration-200 mb-2">
                {{ getText('confirmReviewButton') || 'ПОДТВЕРДИТЬ ПРОСМОТР' }}
              </button>
              <button @click="closeModal"
                class="w-full bg-gray-400 hover:bg-gray-500 text-white font-semibold py-3 px-4 rounded-full transition-colors duration-200">
                {{ getText('cancelButton') || 'ОТМЕНИТЬ' }}
              </button>
              <button @click="showPreview = false"
                class="w-full bg-gray-200 hover:bg-gray-300 text-gray-700 font-semibold py-3 px-4 rounded-full transition-colors duration-200">
                {{ getText('closePreviewButton') || 'Закрыть просмотр' }}
              </button>
            </div>
          </div>
        </div>
      </div>
      </div>
      
      <!-- Final Success Screen (Step 12) - Separate from main questionnaire -->
      <div v-else-if="currentStep === 12" class="w-full bg-white rounded-3xl shadow-md border border-gray-200 p-6 flex flex-col items-center text-center">
        <div class="mb-6">
          <div class="w-16 h-16 bg-green-100 rounded-full flex items-center justify-center mx-auto mb-4">
            <svg class="w-8 h-8 text-green-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"></path>
            </svg>
          </div>
          <h3 class="text-xl font-bold text-gray-800 mb-3">{{ getText('successTitle') || 'Отлично, заявка получена!' }}</h3>
          <p class="text-gray-600 mb-4">{{ getText('successMessage') || 'Скоро мы с вами свяжемся, чтобы всё заработало.' }}</p>
          <div class="text-sm font-medium text-[#FF2D6A] italic">
            {{ getText('brandMessage') || 'Radio Yonar — слышит, замечает, усиливает!' }}
          </div>
        </div>
        <div class="flex justify-center">
          <button @click="closeModal"
            class="bg-[#c0d700] hover:bg-[#a8c000] text-white font-semibold py-3 px-12 rounded-lg transition-colors duration-200">
            {{ getText('closeButton') || 'Закрыть' }}
          </button>
        </div>
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
      showPreview: false,
      hasReviewed: false,
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
      console.log('nextStep called, currentStep:', this.currentStep)
      console.log('canProceed result:', this.canProceed())
      
      if (!this.canProceed()) {
        console.log('Cannot proceed, returning')
        return
      }

      if (this.currentStep === 10) {
        this.currentStep = 11
        console.log('Step incremented to:', this.currentStep)
      }
      else if (this.currentStep === 11) {
        this.currentStep = 12
        console.log('Step incremented to:', this.currentStep)
      }
      else if (this.currentStep < 10) {
        this.currentStep++
        console.log('Step incremented to:', this.currentStep)
      }
    },
    prevStep() {
      if (this.currentStep > 0) {
        this.currentStep--
      }
    },
    markAsReviewed() {
      this.hasReviewed = true
      this.$nextTick(() => {
      })
    },
    submitForm() {
      this.$emit('submit', this.formData)
      this.currentStep = 12
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
          return !!this.formData.businessPhoto
        case 8:
          return !!this.formData.documentPhoto
        case 9:
          return (this.formData.mobilePhone || '').trim().length > 0 && (this.formData.email || '').trim().length > 0
        case 10:
          return !!this.formData.selfieVideo
        case 11:
          return true
        default:
          return true
      }
    },

    handleFileUpload(event, fieldName) {
      const file = event.target.files[0]
      if (!file) return

      const maxSize = 50 * 1024 * 1024
      if (file.size > maxSize) {
        this.fileErrors[fieldName] = 'File size must be 50MB or less'
        this.formData[fieldName] = null
        return
      }

      this.fileErrors[fieldName] = ''
      
      this.formData[fieldName] = file
      this.fileSizes[fieldName] = file.size
    },

    submitForm() {
      this.$emit('submit', this.formData)
      this.currentStep = 11
    },

    editForm() {
      this.showPreview = false
      this.hasReviewed = false
      this.currentStep = 2
    },

    closeModal() {
      this.currentStep = 0
      this.selectedLanguage = ''
      this.currentLanguage = 'ru'
      this.hasReviewed = false
      this.showPreview = false
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

.bg-\[#FF2D6A\], .bg-\[#FF2D6A\]:hover {
  background-color: #c0d700 !important;
}

.hover\:bg-\[#e0265c\]:hover {
  background-color: #a8c000 !important;
}

.focus\:ring-\[#FF2D6A\]:focus {
  --tw-ring-color: #c0d700 !important;
}

.text-\[#FF2D6A\] {
  color: #c0d700 !important;
}

input[type="checkbox"].text-\[#FF2D6A\]:checked {
  background-color: #c0d700 !important;
  border-color: #c0d700 !important;
}
</style>