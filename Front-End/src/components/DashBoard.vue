<script setup lang="ts">
import { ref, watch, onMounted } from 'vue';
import '../assets/dashboard.css';
import { useRouter } from 'vue-router';
import axios from 'axios';


const router = useRouter()
const isNavOpen = ref(true)
const checkRole = ref(true)
const showModal2 = ref(false)
const showModal3 = ref(false)
const bankName = import.meta.env.VITE_NAME_BANK
const bankAccount = import.meta.env.VITE_BANK_ACCOUNT
const ownerAccount = import.meta.env.VITE_OWNER_ACCOUNT
const selectedPrice = ref('')
const paymentNote = ref('')
const qrImg = ref('')
const closeModal2 = () => {
  showModal2.value = false
}

const closeModal3 = () => {
  showModal3.value = false
}

const getQR = async () => {
  try {
    const bankAccount = '0902219942' // Ví dụ tài khoản ngân hàng
    const email = localStorage.getItem('email') || 'guest@example.com'
    const amount = selectedPrice.value.replace(/\D/g, '') // Lấy giá trị tiền từ `selectedPrice` và loại bỏ ký tự không phải số
    
    // Gọi API để tạo QR
    const response = await axios.post('https://api.vietqr.io/v2/generate', {
      accountNo: bankAccount,
      accountName: 'AI kỷ nguyên vươn mình',
      acqId: '970425',
      amount: amount,
      addInfo: email,
      format: 'text',
      template: 'compact'
    })

    // Lấy URL QR và gán vào `qrImg`
    qrImg.value = response.data.data.qrDataURL
    console.log(response.data.data) // In ra URL QR cho kiểm tra
  } catch (error) {
    console.error('Có lỗi khi tạo QR:', error)
  }
}

const goToPayment = async (item: any) => {
  selectedPrice.value = item.price + " " + item.format
  const email = localStorage.getItem('email') || 'guest@example.com'
  paymentNote.value = `${email}`

  await getQR()

  showModal2.value = false
  setTimeout(() => {
    showModal3.value = true
  }, 300)
}

const plans = [
  { price: '25.000',format: 'vnđ' , credits: 1, features: ['Tự động trả lời bằng AI Chatbot', 'Miễn phí tích hợp và training AI', 'Hỏi mọi thứ trên ChatGPT'] },
  { price: '125.000',format: 'vnđ' , credits: 5, features: ['Tự động trả lời bằng AI Chatbot', 'Miễn phí tích hợp và training AI', 'Hỏi mọi thứ trên ChatGPT'] },
  { price: '290.000',format: 'vnđ' , credits: 10, features: ['Tự động trả lời bằng AI Chatbot', 'Miễn phí tích hợp và training AI', 'Hỏi mọi thứ trên ChatGPT'] },
  { price: '1.200.000',format: 'vnđ' , credits: 50, features: ['Tự động trả lời bằng AI Chatbot', 'Miễn phí tích hợp và training AI', 'Hỏi mọi thứ trên ChatGPT'] },
]
// Theo dõi sự thay đổi của checkbox để cập nhật trạng thái
watch(isNavOpen, (newVal) => {

});

onMounted(() => {
  const token = localStorage.getItem('token')
  const role = localStorage.getItem('role')

  if (role === 'admin') {
    checkRole.value = true;// Cập nhật giá trị của checkRole thông qua .value
  } else {
    checkRole.value = false
  }

  if (!token) {
    router.push('/login'); // Chuyển hướng về trang đăng nhập
  }
});

const logout = () => {
  localStorage.clear() // Xóa token khỏi localStorage
  router.push('/login') // Chuyển hướng về trang đăng nhập
};


</script>

<template>
  <div :class="['drawer lg:drawer-open', { 'drawer-open': isNavOpen == true }]">
    <input id="my-drawer" type="checkbox" class="drawer-toggle" v-model="isNavOpen" />
    <div class="drawer-content bg-base-300">
      <!-- Page content here -->
      <div class="navbar bg-base-100 border-b border-base-300">
        <label for="my-drawer" class="btn btn-square btn-ghost btn-hover">
          <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
            <path fill="#000" d="M3 18v-2h18v2zm0-5v-2h18v2zm0-5V6h18v2z" />
          </svg>
        </label>
        <div class="navbar-start">
          <a class="btn btn-ghost normal-case text-xl text-black btn-hover">AIknvm</a>
        </div>
        <div class="navbar-end">
          <button class="btn" @click="showModal2 = true">Gia hạn</button>
        </div>
      </div>
      <router-view></router-view>
    </div>
    <div :class="['drawer-side border-r border-base-300', { 'd-none': isNavOpen == false }]">
      <label for="my-drawer" aria-label="close sidebar" class="drawer-overlay"></label>
      <ul class="menu bg-base-200 text-base-content min-h-full w-80 p-4">
        <li>
          <router-link to="/">
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
              <path fill="currentColor"
                d="M19 9.3V4h-3v2.6L12 3L2 12h3v8h5v-6h4v6h5v-8h3zm-9 .7c0-1.1.9-2 2-2s2 .9 2 2z" />
            </svg>
            Trang chủ
          </router-link>
        </li>
        <div class="divider my-0"></div>
        <li>
          <router-link to="/dashboard/chat">
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
              <path fill="currentColor"
                d="M4 18h2v4.081L11.101 18H16c1.103 0 2-.897 2-2V8c0-1.103-.897-2-2-2H4c-1.103 0-2 .897-2 2v8c0 1.103.897 2 2 2" />
              <path fill="currentColor"
                d="M20 2H8c-1.103 0-2 .897-2 2h12c1.103 0 2 .897 2 2v8c1.103 0 2-.897 2-2V4c0-1.103-.897-2-2-2" />
            </svg>
            Chat
          </router-link>
        </li>
        <li>
          <details open class="dropdown">
            <summary><span
                class="bg-clip-text bg-gradient-to-l from-primary to-secondary text-transparent flex gap-2 items-center">
                <svg data-v-cf1ec82f="" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"
                  aria-hidden="true" role="img" class="icon text-primary w-6 h-6" width="1em" height="1em"
                  viewBox="0 0 24 24">
                  <g fill="none">
                    <path
                      d="m12.594 23.258l-.012.002l-.071.035l-.02.004l-.014-.004l-.071-.036q-.016-.004-.024.006l-.004.01l-.017.428l.005.02l.01.013l.104.074l.015.004l.012-.004l.104-.074l.012-.016l.004-.017l-.017-.427q-.004-.016-.016-.018m.264-.113l-.014.002l-.184.093l-.01.01l-.003.011l.018.43l.005.012l.008.008l.201.092q.019.005.029-.008l.004-.014l-.034-.614q-.005-.019-.02-.022m-.715.002a.02.02 0 0 0-.027.006l-.006.014l-.034.614q.001.018.017.024l.015-.002l.201-.093l.01-.008l.003-.011l.018-.43l-.003-.012l-.01-.01z">
                    </path>
                    <path fill="currentColor"
                      d="M9.107 5.448c.598-1.75 3.016-1.803 3.725-.159l.06.16l.807 2.36a4 4 0 0 0 2.276 2.411l.217.081l2.36.806c1.75.598 1.803 3.016.16 3.725l-.16.06l-2.36.807a4 4 0 0 0-2.412 2.276l-.081.216l-.806 2.361c-.598 1.75-3.016 1.803-3.724.16l-.062-.16l-.806-2.36a4 4 0 0 0-2.276-2.412l-.216-.081l-2.36-.806c-1.751-.598-1.804-3.016-.16-3.724l.16-.062l2.36-.806A4 4 0 0 0 8.22 8.025l.081-.216zM19 2a1 1 0 0 1 .898.56l.048.117l.35 1.026l1.027.35a1 1 0 0 1 .118 1.845l-.118.048l-1.026.35l-.35 1.027a1 1 0 0 1-1.845.117l-.048-.117l-.35-1.026l-1.027-.35a1 1 0 0 1-.118-1.845l.118-.048l1.026-.35l.35-1.027A1 1 0 0 1 19 2">
                    </path>
                  </g>
                </svg> Chatbot AI
              </span></summary>
            <ul tabindex="0" class="menu menu-sm gap-1 w-full pr-4">
              <li>
                <router-link to="/dashboard/chatbox/setting"><svg xmlns="http://www.w3.org/2000/svg" width="24"
                    height="24" viewBox="0 0 24 24">
                    <path fill="currentColor"
                      d="m21.51 14.59l-1.25-1.32a7.9 7.9 0 0 0-.06-2.9l1.22-1.32a.76.76 0 0 0 .14-.79a10.3 10.3 0 0 0-2.2-3.35a.74.74 0 0 0-.72-.19l-1.84.47a8.5 8.5 0 0 0-1.83-1l-.45-1.72a.73.73 0 0 0-.59-.55a10 10 0 0 0-1.89-.17a9.4 9.4 0 0 0-2.35.31a.73.73 0 0 0-.53.53l-.48 1.77a8.2 8.2 0 0 0-1.52.88l-1.82-.45a.73.73 0 0 0-.72.21a10 10 0 0 0-2.23 3.62a.76.76 0 0 0 .16.77l1.26 1.31q-.096.631-.1 1.27c0 .3 0 .6.05.9l-1.31 1.46a.75.75 0 0 0-.16.73a10 10 0 0 0 2 3.59a.75.75 0 0 0 .76.24l1.72-.44a8 8 0 0 0 2 1.23l.5 1.79a.77.77 0 0 0 .56.53c.721.163 1.459.247 2.2.25q.886-.01 1.76-.17a.75.75 0 0 0 .59-.53l.47-1.69a8.1 8.1 0 0 0 2.38-1.34l1.76.4a.74.74 0 0 0 .73-.24a10.1 10.1 0 0 0 2-3.34a.76.76 0 0 0-.21-.75m-9.39 1.27a3.81 3.81 0 1 1-.021-7.619a3.81 3.81 0 0 1 .02 7.62" />
                  </svg>Cấu hình câu trả lời
                </router-link>
              </li>
              <li>
                <router-link to="/dashboard/chatbox/store">
                  <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
                    <path fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round"
                      stroke-width="2"
                      d="M3 18V6.2c0-1.12 0-1.68.218-2.108c.192-.377.497-.682.874-.874C4.52 3 5.08 3 6.2 3h11.6c1.12 0 1.68 0 2.108.218a2 2 0 0 1 .874.874c.218.427.218.987.218 2.105v11.606c0 .485 0 .865-.018 1.174M3 18c0 .988.013 1.506.218 1.907c.192.377.497.683.874.875c.427.218.987.218 2.105.218h11.607c1.118 0 1.677 0 2.104-.218c.376-.192.682-.498.874-.875c.123-.242.177-.526.2-.93M3 18l4.768-5.562l.001-.002c.423-.493.635-.74.886-.83a1 1 0 0 1 .681.005c.25.093.46.343.876.843l2.671 3.205c.386.464.58.696.816.79a1 1 0 0 0 .651.028c.244-.072.46-.287.889-.716l.497-.497c.437-.438.656-.656.904-.728a1 1 0 0 1 .659.037c.238.099.431.34.818.822l2.865 3.582m0 0L21 19M15 9a1 1 0 1 1 0-2a1 1 0 0 1 0 2" />
                  </svg>Kho ảnh và tài liệu
                </router-link>
              </li>
            </ul>
          </details>
        </li>
        <li>
          <router-link to="/dashboard/payment">
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
              <path fill="currentColor" fill-rule="evenodd"
                d="M7.245 2h9.51c1.159 0 1.738 0 2.206.163a3.05 3.05 0 0 1 1.881 1.936C21 4.581 21 5.177 21 6.37v14.004c0 .858-.985 1.314-1.608.744a.946.946 0 0 0-1.284 0l-.483.442a1.657 1.657 0 0 1-2.25 0a1.657 1.657 0 0 0-2.25 0a1.657 1.657 0 0 1-2.25 0a1.657 1.657 0 0 0-2.25 0a1.657 1.657 0 0 1-2.25 0l-.483-.442a.946.946 0 0 0-1.284 0c-.623.57-1.608.114-1.608-.744V6.37c0-1.193 0-1.79.158-2.27c.3-.913.995-1.629 1.881-1.937C5.507 2 6.086 2 7.245 2M7 6.75a.75.75 0 0 0 0 1.5h.5a.75.75 0 0 0 0-1.5zm3.5 0a.75.75 0 0 0 0 1.5H17a.75.75 0 0 0 0-1.5zM7 10.25a.75.75 0 0 0 0 1.5h.5a.75.75 0 0 0 0-1.5zm3.5 0a.75.75 0 0 0 0 1.5H17a.75.75 0 0 0 0-1.5zM7 13.75a.75.75 0 0 0 0 1.5h.5a.75.75 0 0 0 0-1.5zm3.5 0a.75.75 0 0 0 0 1.5H17a.75.75 0 0 0 0-1.5z"
                clip-rule="evenodd" />
            </svg>
            thanh toán và sử dụng
          </router-link>
        </li>
        <div class="divider my-0" v-if="checkRole"></div>
        <li v-if="checkRole">
          <router-link to="/dashboard/list-user">
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
              <circle cx="12" cy="6" r="4" fill="currentColor" />
              <path fill="currentColor" d="M20 17.5c0 2.485 0 4.5-8 4.5s-8-2.015-8-4.5S7.582 13 12 13s8 2.015 8 4.5" />
            </svg>
            Danh sách tài khoản
          </router-link>
        </li>
        <li v-if="checkRole">
          <router-link to="/dashboard/list-bot">
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 28 28">
              <path fill="currentColor"
                d="M14 1.5a.75.75 0 0 1 .75.75V3h4.5A2.75 2.75 0 0 1 22 5.75v5.5A2.75 2.75 0 0 1 19.25 14H8.75A2.75 2.75 0 0 1 6 11.25v-5.5A2.75 2.75 0 0 1 8.75 3h4.5v-.75A.75.75 0 0 1 14 1.5M6.75 16A2.75 2.75 0 0 0 4 18.75v.75c0 1.977.961 3.642 2.717 4.78C8.444 25.397 10.917 26 14 26s5.556-.602 7.283-1.72C23.039 23.141 24 21.476 24 19.5v-.75A2.75 2.75 0 0 0 21.25 16zM11 10a1.5 1.5 0 1 0 0-3a1.5 1.5 0 0 0 0 3m7.5-1.5a1.5 1.5 0 1 0-3 0a1.5 1.5 0 0 0 3 0" />
            </svg>
            Danh sách bot AI
          </router-link>
        </li>
        <li v-if="checkRole">
          <router-link to="/dashboard/list-chat">
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
              <g fill="currentColor" fill-opacity="0" stroke="currentColor" stroke-linecap="round"
                stroke-linejoin="round" stroke-width="1">
                <path stroke-dasharray="12" stroke-dashoffset="12"
                  d="M3.5 5c0 -0.83 0.67 -1.5 1.5 -1.5c0.83 0 1.5 0.67 1.5 1.5c0 0.83 -0.67 1.5 -1.5 1.5c-0.83 0 -1.5 -0.67 -1.5 -1.5Z">
                  <animate fill="freeze" attributeName="fill-opacity" begin="1.7s" dur="0.5s" values="0;1" />
                  <animate fill="freeze" attributeName="stroke-dashoffset" dur="0.2s" values="12;0" />
                </path>
                <path stroke-dasharray="12" stroke-dashoffset="12"
                  d="M3.5 12c0 -0.83 0.67 -1.5 1.5 -1.5c0.83 0 1.5 0.67 1.5 1.5c0 0.83 -0.67 1.5 -1.5 1.5c-0.83 0 -1.5 -0.67 -1.5 -1.5Z">
                  <animate fill="freeze" attributeName="fill-opacity" begin="2.3s" dur="0.5s" values="0;1" />
                  <animate fill="freeze" attributeName="stroke-dashoffset" begin="0.6s" dur="0.2s" values="12;0" />
                </path>
                <path stroke-dasharray="12" stroke-dashoffset="12"
                  d="M3.5 19c0 -0.83 0.67 -1.5 1.5 -1.5c0.83 0 1.5 0.67 1.5 1.5c0 0.83 -0.67 1.5 -1.5 1.5c-0.83 0 -1.5 -0.67 -1.5 -1.5Z">
                  <animate fill="freeze" attributeName="fill-opacity" begin="2.9s" dur="0.5s" values="0;1" />
                  <animate fill="freeze" attributeName="stroke-dashoffset" begin="1.2s" dur="0.2s" values="12;0" />
                </path>
                <path stroke-dasharray="28" stroke-dashoffset="28"
                  d="M9.5 5c0 -0.83 0.67 -1.5 1.5 -1.5h8c0.83 0 1.5 0.67 1.5 1.5c0 0.83 -0.67 1.5 -1.5 1.5h-8c-0.83 0 -1.5 -0.67 -1.5 -1.5Z">
                  <animate fill="freeze" attributeName="fill-opacity" begin="1.9s" dur="0.5s" values="0;1" />
                  <animate fill="freeze" attributeName="stroke-dashoffset" begin="0.2s" dur="0.4s" values="28;0" />
                </path>
                <path stroke-dasharray="28" stroke-dashoffset="28"
                  d="M9.5 12c0 -0.83 0.67 -1.5 1.5 -1.5h8c0.83 0 1.5 0.67 1.5 1.5c0 0.83 -0.67 1.5 -1.5 1.5h-8c-0.83 0 -1.5 -0.67 -1.5 -1.5Z">
                  <animate fill="freeze" attributeName="fill-opacity" begin="2.5s" dur="0.5s" values="0;1" />
                  <animate fill="freeze" attributeName="stroke-dashoffset" begin="0.8s" dur="0.4s" values="28;0" />
                </path>
                <path stroke-dasharray="28" stroke-dashoffset="28"
                  d="M9.5 19c0 -0.83 0.67 -1.5 1.5 -1.5h8c0.83 0 1.5 0.67 1.5 1.5c0 0.83 -0.67 1.5 -1.5 1.5h-8c-0.83 0 -1.5 -0.67 -1.5 -1.5Z">
                  <animate fill="freeze" attributeName="fill-opacity" begin="3.1s" dur="0.5s" values="0;1" />
                  <animate fill="freeze" attributeName="stroke-dashoffset" begin="1.4s" dur="0.4s" values="28;0" />
                </path>
              </g>
            </svg>
            Danh sách tin nhắn
          </router-link>
        </li>
        <div class="divider my-0"></div>
        <li>
          <button @click="logout" to="/login">
            <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24">
              <path fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4m7 14l5-5l-5-5m5 5H9" />
            </svg>
            Đăng xuất
          </button>
        </li>
      </ul>
    </div>
  </div>
  <div>
    <!-- Modal 2 -->
    <Transition name="fade">
      <dialog v-if="showModal2" id="my_modal_2" class="modal" open>
        <div class="modal-box max-w-7xl xl:max-w-[95vw] w-full [&>button.absolute]:z-10 bg-base-200">
          <button @click="closeModal2" class="btn btn-sm btn-circle btn-ghost absolute right-2 top-2">✕</button>
          <section class="">
            <!-- Nội dung bảng giá -->
            <div class="grid gap-3 lg:grid-cols-4 sm:grid-cols-2">
              <div v-for="(item, index) in plans" :key="index"
                class="card shadow bg-base-100 hover:shadow-lg transition-all duration-300">
                <div class="card-body text-center px-3">
                  <h3 class="card-title block">{{ item.price }} {{ item.format }}</h3>
                  <div class="my-3">
                    <p class="text-lg font-semibold text-secondary">{{ item.credits }} credits</p>
                  </div>
                  <ul class="text-left">
                    <li v-for="(feature, idx) in item.features" :key="idx" class="flex space-x-3">
                      <span>✔</span>
                      <span>{{ feature }}</span>
                    </li>
                  </ul>
                  <button @click="goToPayment(item)" class="btn btn-primary mt-5">Đăng ký ngay</button>
                </div>
              </div>
            </div>
            <div class="text-center mt-5">(*) 1 credits = 1 USD</div>
          </section>
        </div>
      </dialog>
    </Transition>

    <!-- Modal 3 -->
    <Transition name="fade">
      <dialog v-if="showModal3" id="my_modal_3" class="modal" open>
        <div class="modal-box max-w-7xl xl:max-w-[95vw] w-full [&>button.absolute]:z-10 bg-base-200">
          <button @click="closeModal3" class="btn btn-sm btn-circle btn-ghost absolute right-2 top-2">✕</button>
          <section class="">
            <div class="px-3">
              <div class="text-center my-5">
                <h2 class="section-title">Thanh toán</h2>
              </div>
              <div class="grid gap-5 md:grid-cols-2">
                <div class="bg-white rounded w-full p-5 shadow-lg">
                  <img :src="qrImg" class="max-w-xs mx-auto" />
                </div>
                <div class="md:order-first flex flex-col space-y-5">
                  <div class="p-5 bg-white rounded shadow">
                    <ul class="text-lg">
                      <li><strong>{{ bankName }}</strong></li>
                      <li>Số tài khoản: <span class="text-info font-mono tracking-wider">{{ bankAccount }}</span></li>
                      <li>Chủ tài khoản: <strong class="text-info">{{ ownerAccount }}</strong></li>
                    </ul>
                  </div>
                  <div class="p-5 bg-white rounded text-center h-full shadow">
                    <p class="text-xl">Chuyển khoản: <span class="text-info font-mono">{{ selectedPrice }}</span></p>
                    <p class="text-xl">Nội dung: <span class="text-info">{{ paymentNote }}</span></p>
                    <div class="divider"></div>
                    <p class="opacity-70">
                      Tài khoản của bạn sẽ được kích hoạt ngay khi chúng tôi nhận được chuyển khoản
                    </p>
                  </div>
                </div>
              </div>
            </div>
          </section>
        </div>
      </dialog>
    </Transition>
  </div>
</template>
