<script setup lang="ts">
import { ref, computed, watch, onMounted, reactive } from 'vue'
import axios from 'axios'
import { useToast } from 'vue-toast-notification';

// Interface dữ liệu người dùng
interface Bot {
  _id: string
  name: string
  createdAt: string
  templateMessage: string
}

// State
const bots = ref<Bot[]>([])
const totalItems = ref(0)
const currentPage = ref(1)
const perPage = ref(10)
const perPageOptions = ref([10, 20, 50, 100])
const loading = ref(false)
const botEdit = ref<Bot | null>(null)
const botDetail = reactive({
  name: '',
  templateMessage: '',
})
const titleModal = ref('')
const toast = useToast();
const urlServer = import.meta.env.VITE_URL_SERVER

const totalPages = computed(() => Math.ceil(totalItems.value / perPage.value))
const visiblePages = computed(() => {
  const pages: number[] = []
  const range = 2
  for (let i = 1; i <= totalPages.value; i++) {
    if (
      i <= range ||
      i > totalPages.value - range ||
      (i >= currentPage.value - range && i <= currentPage.value + range)
    ) {
      pages.push(i)
    }
  }
  return pages
})

const fetchUsers = async () => {
  loading.value = true
  try {
    const response = await axios.get(`https://${urlServer}/list-bot`, {
      params: {
        page: currentPage.value,
        limit: perPage.value
      }
    })
    bots.value = response.data.data
    totalItems.value = response.data.total
  } catch (error) {
    console.error('Lỗi khi lấy dữ liệu:', error)
  } finally {
    loading.value = false
  }
}

const closeModal = () => {
  // Reset dữ liệu
  botDetail.name = ''
  botDetail.templateMessage = ''

  botEdit.value = null

  // Đóng modal
  const modal: any = document.getElementById('modal_bot_detail')
  if (modal?.close) modal.close()
}

const getUser = (data: any) => {
  botEdit.value = data

  botDetail.name = botEdit.value?.name ?? ''
  botDetail.templateMessage = botEdit.value?.templateMessage ?? ''

}

const getUserDetail = async (id: string) => {
  if (id === '0') {
    titleModal.value = 'Thêm mới bot AI'
    botEdit.value = null
    botDetail.name = ''
    botDetail.templateMessage = ''
    const modal: any = document.getElementById('modal_bot_detail')
    if (modal?.showModal) modal.showModal()
  }
  else {
    titleModal.value = 'Cập nhật bot AI'
    const findUser = bots.value.find(x => x._id === id)
    getUser(findUser)
    // Mở modal
    const modal: any = document.getElementById('modal_bot_detail')
    if (modal?.showModal) modal.showModal()
  }
}

const updateUserDetail = async () => {
  if (botEdit.value?._id === undefined) {
    try {
      const response = await axios.post(`https://${urlServer}/registerBot`, {
        name: botDetail.name,
        templateMessage: botDetail.templateMessage
      })
      
      bots.value.unshift(response.data) // thêm vào đầu danh sách
      toast.success('Thêm mới thành công!', {
				position: 'top-right',
				duration: 3000
			});
      // Đóng modal
      const modal: any = document.getElementById('modal_bot_detail')
      if (modal?.close) modal.close()
    } catch (error) {
      toast.error('Lỗi khi lưu dữ liệu!', {
			position: 'top-right',
			duration: 3000
		});
    }
  }
  else {
    try {
      const respone = await axios.put(`https://${urlServer}/update-bot/${botEdit.value?._id}`, {
        ...botDetail
      })
      const findBot = bots.value.findIndex(x => x._id === respone.data._id)
      console.log(respone)
      bots.value[findBot] = respone.data
      toast.success('Cập nhật thành công!', {
				position: 'top-right',
				duration: 3000
			});
      // Đóng modal
      const modal: any = document.getElementById('modal_bot_detail')
      if (modal?.close) modal.close()
    } catch (error) {
      toast.error('Lỗi khi lưu dữ liệu!', {
			position: 'top-right',
			duration: 3000
		});
    }
  }
}

watch([currentPage, perPage], fetchUsers)
onMounted(fetchUsers)

const changePerPage = () => {
  currentPage.value = 1
}
</script>

<template>
  <div class="lg:px-10 pb-10 pt-10 min-h-[calc(100vh-5rem)]">
    <div class="p-4 bg-base-200 shadow">
      <div class="flex justify-end">
        <button class="btn" @click="getUserDetail('0')">
          Thêm mới
        </button>
      </div>
      <div class="">
        <div v-if="loading" class="text-center py-5 text-lg font-semibold text-gray-500">
          Đang tải dữ liệu...
        </div>

        <div v-else class="overflow-x-auto">
          <table class="table table-auto">
            <thead>
              <tr>
                <th>#</th>
                <th>Tên</th>
                <th>Ngày tạo</th>
                <th></th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(user, index) in bots" :key="index">
                <td>{{ (currentPage - 1) * perPage + index + 1 }}</td>
                <td>{{ user.name }}</td>
                <td>{{ user.createdAt }}</td>
                <td>
                  <button class="btn btn-ghost btn-xs" @click="getUserDetail(user._id)">
                    <!-- icon giữ nguyên -->
                    <svg xmlns="http://www.w3.org/2000/svg" width="20px" height="20px" viewBox="0 0 512 512">
                      <path fill="currentColor"
                        d="M441 58.9L453.1 71c9.4 9.4 9.4 24.6 0 33.9L424 134.1L377.9 88L407 58.9c9.4-9.4 24.6-9.4 33.9 0zM209.8 256.2L344 121.9l46.1 46.1l-134.3 134.2c-2.9 2.9-6.5 5-10.4 6.1L186.9 325l16.7-58.5c1.1-3.9 3.2-7.5 6.1-10.4zM373.1 25L175.8 222.2c-8.7 8.7-15 19.4-18.3 31.1l-28.6 100c-2.4 8.4-.1 17.4 6.1 23.6s15.2 8.5 23.6 6.1l100-28.6c11.8-3.4 22.5-9.7 31.1-18.3L487 138.9c28.1-28.1 28.1-73.7 0-101.8L474.9 25c-28.1-28.1-73.7-28.1-101.8 0M88 64c-48.6 0-88 39.4-88 88v272c0 48.6 39.4 88 88 88h272c48.6 0 88-39.4 88-88V312c0-13.3-10.7-24-24-24s-24 10.7-24 24v112c0 22.1-17.9 40-40 40H88c-22.1 0-40-17.9-40-40V152c0-22.1 17.9-40 40-40h112c13.3 0 24-10.7 24-24s-10.7-24-24-24z" />
                    </svg>
                  </button>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <!-- Pagination controls -->
        <!-- Pagination controls -->
        <div class="flex justify-between items-center mt-4" v-if="totalItems > 0">
          <div class="join">
            <button class="join-item btn" :disabled="currentPage === 1" @click="currentPage--">
              Prev
            </button>

            <button v-if="currentPage > 3" class="join-item btn" @click="currentPage = 1">1</button>
            <span v-if="currentPage > 3" class="join-item btn">...</span>

            <button v-for="page in visiblePages" :key="page" class="join-item btn"
              :class="{ 'btn-active': page === currentPage }" @click="currentPage = page">
              {{ page }}
            </button>

            <span v-if="currentPage < totalPages - 2" class="join-item btn">...</span>
            <button v-if="currentPage < totalPages - 2" class="join-item btn" @click="currentPage = totalPages">
              {{ totalPages }}
            </button>

            <button class="join-item btn" :disabled="currentPage === totalPages" @click="currentPage++">
              Next
            </button>
          </div>

          <!-- Chọn số bản ghi / trang -->
          <select v-model="perPage" @change="changePerPage" class="select select-bordered">
            <option v-for="option in perPageOptions" :key="option" :value="option">{{ option }}</option>
          </select>
        </div>

        <!-- Nếu không có dữ liệu -->
        <div v-else class="text-center mt-4 text-gray-500">
          Không có dữ liệu để hiển thị.
        </div>

      </div>
    </div>
  </div>

  <dialog id="modal_bot_detail" class="modal" v-if="botDetail">
    <div class="modal-box max-w-7xl xl:max-w-[95vw] w-full bg-base-200">
      <div class="flex justify-between card-title">
        <h1>
          {{ titleModal }}
        </h1>
        <button class="btn btn-sm btn-circle btn-ghost absolute right-2 top-2" @click="closeModal">✕</button>
      </div>
      <section>
        <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mb-4">
          <div>
            <label class="label">Tên</label>
            <input type="text" class="input input-bordered w-full" v-model="botDetail.name" />
          </div>
          <div>
            <label class="label">Mẫu tin nhắn</label>
            <textarea type="text" class="input input-bordered w-full" v-model="botDetail.templateMessage"></textarea>
          </div>
          <div class="flex justify-end gap-2 mt-4">
            <button class="btn" @click="closeModal">Hủy</button>
            <button class="btn btn-primary" @click="updateUserDetail">Lưu</button>
          </div>
        </div>
      </section>
    </div>
    <form method="dialog" class="modal-backdrop">
      <button>close</button>
    </form>
  </dialog>


</template>

<style>
thead,
tfoot {
  color: black;
}

.join button {
  background-color: #f4f4f4;
  color: black;
}

.btn-ghost:hover {
  background-color: unset;
  color: black
}

.select {
  width: 65px;
  background-color: white;
  border: 1px black solid;
}

.modal input,
.modal textarea {
  background-color: #f4f4f4;
  color: black;
}
</style>