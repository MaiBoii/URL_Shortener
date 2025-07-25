<template>
  <main>
    <h1>QR 코드 생성기</h1>
    <p>목적에 맞는 QR 코드를 쉽게 생성해보세요.</p>
    <div class="flex-container">
      <div class="form">
        <div class="type-selector">
          <div
            v-for="option in options"
            :key="option.value"
            class="type-option"
            :class="{ active: type === option.value }"
            @click="type = option.value"
          >
            <i :class="option.icon"></i> {{ option.label }}
          </div>
        </div>

        <form class="inputs" @submit.prevent>
          <template v-if="type === 'text' || type === 'link'">
            <FloatingInput v-model="text" :label="type === 'text' ? '텍스트 입력' : 'URL 입력'" />
          </template>

          <template v-else-if="type === 'wifi'">
            <FloatingInput v-model="ssid" label="SSID 입력" />
            <FloatingInput v-model="password" label="비밀번호 입력" />
          </template>

          <template v-else-if="type === 'sms'">
            <FloatingInput v-model="phone" label="전화번호 입력" />
            <FloatingInput v-model="smsMessage" label="메시지 입력" />
          </template>

          <template v-else-if="type === 'email'">
            <FloatingInput v-model="email" label="이메일 주소 입력" />
            <FloatingInput v-model="emailSubject" label="제목 입력" />
            <FloatingInput v-model="emailBody" label="내용 입력" />
          </template>

          <template v-else-if="type === 'x'">
            <RadioGroup v-model="twitterMode" :options="twitterOptions" />
            <FloatingInput v-model="twitterUsername" label="Twitter 사용자 이름 입력" />

            <template v-if="twitterMode === 'tweet'">
              <FloatingInput v-model="tweetId" label="트윗 ID 입력" />
            </template>
          </template>
        </form>
      </div>

      <div class="qrcode-section">
        <div class="qrcode-container" ref="dashboardRef">
          <qrcode-vue :value="finalText || ''" :size="200" />
        </div>
        <div class="download-button-wrapper">
          <button @click="downloadDashboard">이미지 다운로드</button>
        </div>
      </div>
    </div>
  </main>
</template>


<script setup lang="ts">
import { ref, computed } from 'vue'
import QrcodeVue from 'qrcode.vue'
import html2canvas from 'html2canvas'
import FloatingInput from './FloatingInput.vue'
import RadioGroup from './RadioGroup.vue'


// 타입 옵션들
const options = [
  { value: 'text', label: '텍스트', icon: 'fas fa-note-sticky' },
  { value: 'link', label: '링크', icon: 'fas fa-globe' },
  { value: 'wifi', label: 'Wi-Fi', icon: 'fas fa-wifi' },
  { value: 'sms', label: 'SMS', icon: 'fas fa-sms' },
  { value: 'email', label: '이메일', icon: 'fas fa-envelope' },
  { value: 'x', label: 'Twitter', icon: 'fa-brands fa-x-twitter' },
]

// 상태 관리
const type = ref('text')
const text = ref('')
const link = ref('')
const ssid = ref('')
const password = ref('')
const phone = ref('')
const smsMessage = ref('')
const email = ref('')
const emailSubject = ref('')
const emailBody = ref('')
const twitterMode = ref<'profile' | 'tweet'>('profile')
const twitterUsername = ref('')
const tweetId = ref('')
const twitterOptions = [
  { label: '프로필', value: 'profile' },
  { label: '트윗', value: 'tweet' },
]

const dashboardRef = ref<HTMLElement | null>(null)

const finalText = computed<string>(() => {
  switch (type.value) {
    case 'text':
      return text.value
    case 'link':
      return link.value
    case 'wifi':
      return ssid.value && password.value ? `WIFI:T:WPA;S:${ssid.value};P:${password.value};;` : ''
    case 'sms':
      return phone.value && smsMessage.value ? `SMSTO:${phone.value}:${smsMessage.value}` : ''
    case 'email':
      return email.value && emailSubject.value && emailBody.value
        ? `mailto:${email.value}?subject=${encodeURIComponent(emailSubject.value)}&body=${encodeURIComponent(emailBody.value)}`
        : ''
    case 'x':
      return twitterUsername.value && tweetId.value
        ? `https://twitter.com/${twitterUsername.value}/status/${tweetId.value}`
        : twitterUsername.value
          ? `https://twitter.com/${twitterUsername.value}`
          : ''
    default:
      return ''
  }
})

// 이미지 다운로드 기능
const downloadDashboard = async () => {
  if (dashboardRef.value) {
    const canvas = await html2canvas(dashboardRef.value)
    const link = document.createElement('a')
    link.download = 'qrcode.png'
    link.href = canvas.toDataURL('image/png')
    link.click()
  }
}
</script>

<style scoped>

label 				 {
  color:#999; 
  font-size:18px;
  font-weight:normal;
  position:absolute;
  pointer-events:none;
  left:5px;
  top:10px;
  transition:0.2s ease all; 
  -moz-transition:0.2s ease all; 
  -webkit-transition:0.2s ease all;
}

input 				{
  font-size:18px;
  padding:10px 10px 10px 5px;
  display:block;
  width:500px;
  border:none;
  margin: 16px; /* 간격 조절 */
  margin-bottom: 30px; /* 간격 조절 */
  border-bottom: 1px solid var(--clr-border);
  background: transparent;
  color: inherit;
}

input:focus 		{
   outline:none; 
}

/* active state */
input:focus ~ label, input:valid ~ label 		{
  top:-20px;
  font-size:14px;
  color:#5264AE;
}


.flex-container {
  display: flex;
  gap: 2rem; /* 요소 간 간격 조절 */
  align-items: flex-start; /* 상단 정렬 */
  max-width: 1000px;   /* 너비 제한 */
  width: 100%;
  margin: 0 auto;      /* 가운데 정렬 */
  padding: 1rem;       /* 양 옆 여백 */
}

.form {
  display: flex;
  flex: 1;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
  margin-bottom: 2rem;
}

.type-selector {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
  justify-content: center;
}

.type-option {
  padding: 0.5rem 1rem;
  border: 1px solid var(--clr-option-bd);
  border-radius: 10px;
  cursor: pointer;
  background-color: var(--clr-surface);
  display: flex;
  align-items: center;
  gap: 0.5rem;
  transition: 0.2s;
}

.type-option:hover {
  background-color: var(--clr-surface-hover);
}

.type-option.active {
  background-color: var(--clr-primary);
  color: var(--clr-on-primary);
}

.highlight {
  position:absolute;
  height:60%; 
  width:100px; 
  top:25%; 
  left:0;
  pointer-events:none;
  opacity:0.5;
}

.type-option.active {
  background-color: #4f46e5;
  color: white;
}


button {
  padding: 0.5rem 1rem;
  font-weight: bold;
  background-color: #4f46e5;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.qrcode-section {
  display: flex;
  flex-direction: column; /* 수직 정렬 */
  align-items: center;
  gap: 1rem;
}

.download-button-wrapper {
  text-align: center;
}

.qrcode-container {
  width: 250px;
  height: 250px;
  margin: 0 auto;
  padding: 1rem;
  border: 2px solid #ccc;
  border-radius: 10px;
  background-color: #f9f9f9;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  position: relative;
  flex-shrink: 0; /* 줄어들지 않도록 */
}

.logo {
  position: absolute;
  width: 40px;
  height: 40px;
}
</style>
