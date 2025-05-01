<template>
  <div class="space-y-4 md:space-y-6 p-4 md:p-6">
    <!-- 페이지 헤더 -->
    <div class="flex flex-col md:flex-row justify-between items-start md:items-center gap-4">
      <div>
        <p class="text-sm text-gray-500 mt-1">작업 현황과 일정을 확인할 수 있습니다.</p>
      </div>
      <div class="flex flex-col md:flex-row gap-3 w-full md:w-auto">
        <div class="relative w-full md:w-64">
          <input
            type="text"
            v-model="searchQuery"
            @input="handleInput"
            placeholder="고객명 또는 주소로 검색"
            class="w-full pl-10 pr-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 bg-white" />
          <i class="fas fa-search absolute left-3 top-1/2 transform -translate-y-1/2 text-gray-400"></i>
        </div>
        <div class="flex flex-col sm:flex-row gap-2 w-full md:w-auto">
          <select
            v-model="stausFilter"
            class="w-full sm:w-auto px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 bg-white">
            <option value="all">전체 상태</option>
            <option value="pending">대기중</option>
            <option value="in_progress">진행중</option>
            <option value="completed">완료</option>
          </select>
          <select
            class="w-full sm:w-auto px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 bg-white">
            <option value="all">전체 기간</option>
            <option value="today">오늘</option>
            <option value="week">이번 주</option>
            <option value="month">이번 달</option>
          </select>
        </div>
      </div>
    </div>
    <!-- 작업목록 -->
    <div class="bg-white rounded-lg shadow-sm">
      <div class="p-4 border-b border-gray-200">
        <h2 class="text-lg font-semibold text-gray-800">배정된 예약 목록</h2>
      </div>
      <!-- 모바일 카드 뷰 -->
      <!-- 모바일 카드 뷰 -->
      <div class="md:hidden space-y-4 p-4">
        <div
          v-for="job in paginatedJobs"
          :key="job.id"
          class="bg-white border rounded-lg p-4 space-y-4 hover:bg-gray-50 transition-all duration-200">
          <div class="flex justify-between items-start">
            <div class="space-y-2">
              <span :class="['px-3 py-1 rounded-full sm:text-sm  text-xs font-medium', getStatusClass(job.status)]">
                <i :class="getStatusIcon(job.status)" class="mr-1"></i>
                {{ getStatusText(job.status) }}
              </span>
              <!--priority 우선순위 -->
              <span
                :class="[
                  'px-2 py-0.5 text-xs font-medium rounded-full',
                  job.priority === '높음'
                    ? 'bg-red-100 text-red-800'
                    : job.priority === '보통'
                    ? 'bg-yellow-100 text-yellow-800'
                    : 'bg-green-100 text-green-800',
                ]">
                {{ job.priority }}
              </span>
            </div>
            <div class="text-right">
              <div class="text-sm text-gray-900">
                {{ formatDate(job.date) }}
              </div>
              <div class="text-sm text-gray-500">{{ job.time }}</div>
            </div>
          </div>

          <div class="space-y-2">
            <div class="text-sm font-medium text-gray-900">
              {{ job.customer }}
            </div>
            <div class="text-sm text-gray-500">
              <i class="fas fa-phone mr-1"></i>{{ job.contact?.primary || job.phone }}
            </div>
            <div class="text-sm text-gray-500"><i class="fas fa-map-marker-alt mr-1"></i>{{ job.address }}</div>
          </div>

          <div class="space-y-2">
            <div class="text-sm font-medium text-gray-900">
              {{ job.serviceType }}
            </div>
            <div class="text-sm text-gray-500">{{ job.duration }}</div>
            <div class="text-sm text-gray-500"><i class="fas fa-users mr-1"></i>{{ job.partySize }}명</div>
          </div>

          <div class="flex flex-wrap gap-2 pt-2">
            <button
              v-if="job.status !== 'completed'"
              @click="startJob(job)"
              class="flex-1 px-3 py-1.5 bg-indigo-600 text-white rounded-md hover:bg-indigo-700 transition-colors duration-200">
              <i class="fas fa-play mr-1"></i>시작
            </button>
            <button
              v-if="job.status === 'in_progress'"
              @click="completeJob(job)"
              class="flex-1 px-3 py-1.5 bg-green-600 text-white rounded-md hover:bg-green-700 transition-colors duration-200">
              <i class="fas fa-check mr-1"></i>완료
            </button>
            <button
              @click="showJobDetails(job)"
              class="flex-1 px-3 py-1.5 bg-gray-600 text-white rounded-md hover:bg-gray-700 transition-colors duration-200">
              <i class="fas fa-eye mr-1"></i>상세
            </button>
          </div>
        </div>
      </div>

      <!-- 데스크탑 테이블 뷰 -->
      <div class="hidden md:block overflow-x-auto">
        <table class="min-w-full divide-y divide-gray-200">
          <thead class="bg-gray-50">
            <tr>
              <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">상태</th>
              <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">고객 정보</th>
              <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                서비스 정보
              </th>
              <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">일정</th>
              <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">장소 정보</th>
              <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">액션</th>
            </tr>
          </thead>
          <tbody class="bg-white divide-y divide-gray-200">
            <tr v-for="job in paginatedJobs" :key="job.id" class="hover:bg-gray-50 transition-colors duration-200">
              <td class="px-6 py-4 whitespace-nowrap">
                <span :class="[getStatusClass(job.status), 'px-3 py-1 rounded-full text-sm font-medium']">
                  <i class="mr-1" :class="getStatusIcon(job.status)"></i>
                  {{ getStatusText(job.status) }}
                </span>
                <div class="mt-2">
                  <span
                    :class="[
                      'px-2 py-0.5 text-xs font-medium rounded-full',
                      job.priority === '높음'
                        ? 'bg-red-100 text-red-800'
                        : job.priority === '보통'
                        ? 'bg-yellow-100 text-yellow-800'
                        : 'bg-green-100 text-green-800',
                    ]">
                    {{ job.priority }}
                  </span>
                </div>
              </td>
              <td class="px-6 py-4">
                <div class="text-sm font-medium text-gray-900">
                  {{ job.customer }}
                </div>
                <div class="text-sm text-gray-500">
                  <i class="fas fa-phone mr-1"></i>{{ job.contact?.primary || job.phone }}
                </div>
                <div class="text-sm text-gray-500 mt-1">
                  <i class="fas fa-star text-yellow-400 mr-1"></i>{{ job.customerRating }}
                </div>
              </td>
              <td class="px-6 py-4">
                <div class="text-sm font-medium text-gray-900">
                  {{ job.serviceType }}
                </div>
                <div class="text-sm text-gray-500">{{ job.duration }}</div>
                <div class="text-sm text-gray-500 mt-1">
                  <span
                    :class="[
                      'px-2 py-0.5 text-xs font-medium rounded-full',
                      job.paymentStatus === '결제완료'
                        ? 'bg-green-100 text-green-800'
                        : 'bg-yellow-100 text-yellow-800',
                    ]">
                    {{ job.paymentStatus }}
                  </span>
                </div>
              </td>
              <td class="px-6 py-4 whitespace-nowrap">
                <div class="text-sm text-gray-900">
                  {{ formatDate(job.date) }}
                </div>
                <div class="text-sm text-gray-500">{{ job.time }}</div>
                <div class="text-sm text-gray-500 mt-1"><i class="fas fa-users mr-1"></i>{{ job.partySize }}명</div>
              </td>
              <td class="px-6 py-4">
                <div class="text-sm text-gray-900">
                  {{ job.location?.type || "정보 없음" }}
                </div>
                <div class="text-sm text-gray-500">
                  {{ job.location?.floor || "정보 없음" }}
                </div>
                <div class="text-sm text-gray-500 mt-1">
                  <i class="fas fa-map-marker-alt mr-1"></i>{{ job.address }}
                </div>
              </td>
              <td class="px-6 py-4 whitespace-nowrap text-sm font-medium">
                <div class="flex flex-col space-y-2">
                  <button
                    v-if="job.status !== 'completed'"
                    @click="startJob(job)"
                    class="text-indigo-600 hover:text-indigo-900 flex items-center transition-colors duration-200">
                    <i class="fas fa-play mr-1"></i>시작
                  </button>
                  <button
                    @click="completeJob(job)"
                    v-if="job.status === 'in_progress'"
                    class="text-green-600 hover:text-green-900 flex items-center transition-colors duration-200">
                    <i class="fas fa-check mr-1"></i>완료
                  </button>
                  <button
                    @click="showJobDetails(job)"
                    class="text-blue-600 hover:text-blue-900 flex items-center transition-colors duration-200">
                    <i class="fas fa-eye mr-1"></i>상세
                  </button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    <!-- 페이지네이션 -->
    <div class="flex flex-col sm:flex-row justify-between items-center gap-4 bg-white rounded-lg shadow-sm p-4">
      <div class="text-sm text-gray-700">
        총 <span class="font-medium">{{ filteredJobs.length }}</span
        >건의 예약
      </div>
      <div class="flex gap-2">
        <button
          @click="prevPage"
          :disabled="currentPage === 1"
          class="px-3 py-1.5 border border-gray-300 rounded hover:bg-gray-50 disabled:opacity-50 disabled:cursor-not-allowed transition-colors duration-200">
          <i class="fas fa-chevron-left"></i>
        </button>
        <div class="flex gap-1">
          <button
            v-for="page in totalPages"
            :key="page"
            @click="goToPage(page)"
            :class="[
              currentPage === page ? 'bg-indigo-600 text-white border-indigo-600' : 'border-gray-300 hover:bg-gray-50',
            ]"
            class="px-3 py-1.5 border rounded transition-colors duration-200">
            {{ page }}
          </button>
        </div>
        <button
          @click="nextPage"
          :disabled="currentPage === totalPages"
          class="px-3 py-1.5 border border-gray-300 rounded hover:bg-gray-50 disabled:opacity-50 disabled:cursor-not-allowed transition-colors duration-200">
          <i class="fas fa-chevron-right"></i>
        </button>
      </div>
    </div>
    <!-- 작업 상세 모달 -->
    <div
      v-if="selectedJob"
      @click="closeModal"
      class="fixed inset-0 bg-gray-500 bg-opacity-75 flex items-center justify-center p-4 z-50">
      <div class="bg-white rounded-lg shadow-xl max-w-2xl w-full max-h-[90vh] overflow-y-auto" @click.stop>
        <div class="p-6 border-b border-gray-200 flex justify-between items-center">
          <h2 class="text-xl font-semibold text-gray-900">작업 상세 정보</h2>
          <button @click="closeModal" class="text-gray-400 hover:text-gray-500 transition-colors duration-200">
            <i class="fas fa-times"></i>
          </button>
        </div>

        <div class="p-6 space-y-6">
          <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4">
            <div class="flex flex-wrap items-center gap-2">
              <span :class="['px-3 py-1 rounded-full text-sm font-medium', getStatusClass(selectedJob.status)]">
                {{ getStatusText(selectedJob.status) }}
              </span>
              <span
                :class="[
                  'px-2 py-0.5 text-xs font-medium rounded-full',
                  selectedJob.priority === '높음'
                    ? 'bg-red-100 text-red-800' //'높음'이면 빨간 배경 + 빨간 글씨
                    : selectedJob.priority === '보통'
                    ? 'bg-yellow-100 text-yellow-800' // '보통'이면 노란 배경 + 노란 글씨
                    : 'bg-green-100 text-green-800', // 그 외(즉, '낮음')이면 초록 배경 + 초록 글씨
                ]">
                {{ selectedJob.priority }}
              </span>
            </div>
            <span class="text-sm text-gray-500"> {{ formatDate(selectedJob.date) }} {{ selectedJob.time }}</span>
          </div>

          <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
            <!-- 고객 정보 섹션 -->
            <div class="space-y-4">
              <h3 class="text-lg font-medium text-gray-900">고객 정보</h3>
              <div class="bg-gray-50 p-4 rounded-lg space-y-3">
                <div>
                  <label class="block text-sm text-gray-500 mb-1">이름</label>
                  <p class="text-gray-900">{{ selectedJob.customer }}</p>
                </div>
                <div>
                  <label class="block text-sm text-gray-500 mb-1">연락처</label>
                  <p class="text-gray-900">{{ selectedJob.phone }}</p>
                </div>
                <div>
                  <label class="block text-sm text-gray-500 mb-1">주소</label>
                  <p class="text-gray-900">{{ selectedJob.addres }}</p>
                </div>
                <div>
                  <label class="block text-sm text-gray-500 mb-1">인원</label>
                  <p class="text-gray-900">{{ selectedJob.partySize }}명</p>
                </div>
                <div>
                  <label class="block text-sm text-gray-500 mb-1">고객 평점</label>
                  <div class="flex items-center">
                    <i class="fas fa-star text-yellow-400 mr-1"></i>
                    <span class="text-gray-900">{{ selectedJob.customerRating }}</span>
                  </div>
                </div>
              </div>
            </div>

            <!-- 서비스 정보 섹션 -->
            <div class="space-y-4">
              <h3 class="text-lg font-medium text-gray-900">서비스 정보</h3>
              <div class="bg-gray-50 p-4 rounded-lg space-y-3">
                <div>
                  <label class="block text-sm text-gray-500 mb-1">서비스 유형</label>
                  <p class="text-gray-900">{{ selectedJob.serviceType }}</p>
                </div>
                <div>
                  <label class="block text-sm text-gray-500 mb-1">작업 시간</label>
                  <p class="text-gray-900">{{ selectedJob.duration }}</p>
                </div>
                <div>
                  <label class="block text-sm text-gray-500 mb-1">담당 작업자</label>
                  <p class="text-gray-900">{{ selectedJob.assignedWorker }}</p>
                </div>
                <div>
                  <label class="block text-sm text-gray-500 mb-1">결제 상태</label>
                  <span
                    :class="[
                      'px-2 py-0.5 text-xs font-medium rounded-full',
                      selectedJob.paymentStatus === '결제완료'
                        ? 'bg-green-100 text-green-800'
                        : 'bg-yellow-100 text-yellow-800',
                    ]">
                    {{ selectedJob.paymentStatus }}
                  </span>
                </div>
                <div>
                  <label class="block text-sm text-gray-500 mb-1">총 금액</label>
                  <p class="text-gray-900">
                    {{ formatCurrency(selectedJob.totalAmount) }}
                  </p>
                </div>
              </div>
            </div>

            <!-- 특별 요청사항 섹션 -->
            <div class="md:col-span-2 space-y-4">
              <h3 class="text-lg font-medium text-gray-900">특별 요청사항</h3>
              <div class="bg-gray-50 p-4 rounded-lg">
                <div v-if="selectedJob.notes" class="mb-4">
                  <label class="block text-sm text-gray-500 mb-1">메모</label>
                  <p class="text-gray-600">{{ selectedJob.notes }}</p>
                </div>
                <div v-if="selectedJob.specialRequests && selectedJob.specialRequests.length > 0">
                  <label class="block text-sm text-gray-500 mb-1">요청사항</label>
                  <div class="flex flex-wrap gap-2">
                    <span
                      v-for="request in selectedJob.specialRequests"
                      :key="request"
                      class="px-2 py-1 bg-blue-100 text-blue-800 rounded-full text-sm">
                      {{ request }}
                    </span>
                  </div>
                </div>
              </div>
            </div>

            <!-- 필요 장비 섹션 -->
            <div class="md:col-span-2 space-y-4">
              <h3 class="text-lg font-medium text-gray-900">필요 장비</h3>
              <div class="bg-gray-50 p-4 rounded-lg">
                <div class="flex flex-wrap gap-2">
                  <span
                    v-for="item in selectedJob.equipment"
                    :key="item"
                    class="px-2 py-1 bg-purple-100 text-purple-800 rounded-full text-sm">
                    {{ item }}
                  </span>
                </div>
              </div>
            </div>

            <!-- 작업 진행 상황 섹션 -->
            <div class="md:col-span-2 space-y-4">
              <h3 class="text-lg font-medium text-gray-900">작업 진행 상황</h3>
              <div class="relative pl-8">
                <div class="absolute left-0 top-0 bottom-0 w-0.5 bg-gray-200"></div>

                <div class="relative pb-8">
                  <div class="absolute left-0 top-0 w-4 h-4 rounded-full border-2 border-white bg-gray-200 -ml-2"></div>
                  <div class="pl-4">
                    <h4 class="font-medium text-gray-900">작업 시작</h4>
                    <p class="text-sm text-gray-500">
                      {{ selectedJob.startTime ? formatTime(selectedJob.startTime) : "아직 시작하지 않음" }}
                    </p>
                  </div>
                </div>

                <div class="relative">
                  <div
                    :class="[
                      'absolute left-0 top-0 w-4 h-4 rounded-full border-2 border-white -ml-2',
                      selectedJob.status === 'completed' ? 'bg-green-500' : 'bg-gray-200',
                    ]"></div>
                  <div class="pl-4">
                    <h4 class="font-medium text-gray-900">작업 완료</h4>
                    <p class="text-sm text-gray-500">
                      {{ selectedJob.completeTime ? formatTime(selectedJob.completeTime) : "아직 시작하지 않음" }}
                    </p>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="p-6 border-t border-gray-200 flex justify-end gap-3">
          <button
            @click="closeModal"
            class="px-4 py-2 bg-gray-600 text-white rounded-lg hover:bg-gray-700 transition-colors duration-200">
            닫기
          </button>
          <button
            v-if="selectedJob.status === 'in_progress'"
            @click="completeJob(selectedJob)"
            class="px-4 py-2 bg-green-600 text-white rounded-lg hover:bg-green-700 transition-colors duration-200">
            완료
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
<script setup>
import { ref, computed } from "vue";
const stausFilter = ref("all");
const searchQuery = ref("");
// 작업목록
const assignedJobs = ref([
  {
    id: 1,
    customer: "김철수",
    phone: "010-1234-5678",
    address: "서울 강남구 테헤란로 123",
    date: "2024-03-20",
    time: "14:00",
    partySize: 4,
    status: "pending",
    notes: "고객이 알레르기가 있으니 주의해주세요.",
    serviceType: "정기청소",
    duration: "2시간",
    priority: "높음",
    paymentStatus: "결제완료",
    totalAmount: 150000,
    startTime: null,
    completeTime: null,
    assignedWorker: "이지은",
    customerRating: 4.8,
    specialRequests: ["창문 청소", "카펫 청소"],
    equipment: ["진공청소기", "스팀청소기", "창문청소기"],
    location: {
      type: "아파트",
      floor: "15층",
      area: "84㎡",
      parking: "지하주차장",
      elevator: true,
    },
    contact: {
      primary: "010-1234-5678",
      secondary: "010-8765-4321",
      emergency: "010-1111-2222",
    },
    schedule: {
      arrivalTime: "13:45",
      estimatedDuration: "2시간",
      bufferTime: "30분",
    },
    requirements: {
      specialAccess: "관리사무소 통과 필요",
      parkingInfo: "지하 2층 B구역",
      entryCode: "1234#",
    },
  },
  {
    id: 2,
    customer: "이영희",
    phone: "010-8765-4321",
    address: "서울 마포구 서강로 456",
    date: "2024-03-20",
    time: "16:30",
    partySize: 2,
    status: "in_progress",
    startTime: "2024-03-20T16:30:00",
    notes: "주차 공간이 협소하니 주의해주세요.",
    serviceType: "이사청소",
    duration: "3시간",
    priority: "보통",
    paymentStatus: "결제완료",
    totalAmount: 200000,
    completeTime: null,
    assignedWorker: "최윤호",
    customerRating: 4.5,
    specialRequests: ["냉장고 청소", "욕실 소독"],
    equipment: ["진공청소기", "스팀청소기", "소독기"],
    location: {
      type: "오피스텔",
      floor: "8층",
      area: "45㎡",
      parking: "노상주차",
      elevator: true,
    },
    contact: {
      primary: "010-8765-4321",
      secondary: "010-1234-5678",
      emergency: "010-3333-4444",
    },
    schedule: {
      arrivalTime: "16:15",
      estimatedDuration: "3시간",
      bufferTime: "45분",
    },
    requirements: {
      specialAccess: "경비실 통과 필요",
      parkingInfo: "건물 앞 노상주차",
      entryCode: "5678#",
    },
  },
  {
    id: 3,
    customer: "박민수",
    phone: "010-1111-2222",
    address: "서울 송파구 올림픽로 789",
    date: "2024-03-21",
    time: "11:00",
    partySize: 6,
    status: "completed",
    startTime: "2024-03-21T11:00:00",
    completeTime: "2024-03-21T12:30:00",
    notes: "특별한 요청사항 없음",
    serviceType: "입주청소",
    duration: "4시간",
    priority: "높음",
    paymentStatus: "결제완료",
    totalAmount: 250000,
    assignedWorker: "정지은",
    customerRating: 4.9,
    specialRequests: ["전체 소독", "창문 청소"],
    equipment: ["진공청소기", "스팀청소기", "소독기", "창문청소기"],
    location: {
      type: "빌라",
      floor: "3층",
      area: "120㎡",
      parking: "전용주차장",
      elevator: false,
    },
    contact: {
      primary: "010-1111-2222",
      secondary: "010-5555-6666",
      emergency: "010-7777-8888",
    },
    schedule: {
      arrivalTime: "10:45",
      estimatedDuration: "4시간",
      bufferTime: "1시간",
    },
    requirements: {
      specialAccess: "현관 비밀번호 필요",
      parkingInfo: "전용주차장 3번",
      entryCode: "9012#",
    },
  },
  {
    id: 4,
    customer: "최지영",
    phone: "010-3333-4444",
    address: "서울 서초구 서초대로 101",
    date: "2024-03-21",
    time: "13:30",
    partySize: 3,
    status: "pending",
    notes: "반려동물이 있으니 주의해주세요.",
    serviceType: "정기청소",
    duration: "2시간",
    priority: "보통",
    paymentStatus: "결제완료",
    totalAmount: 150000,
    startTime: null,
    completeTime: null,
    assignedWorker: "김동현",
    customerRating: 4.7,
    specialRequests: ["애완동물 털 제거"],
    equipment: ["진공청소기", "스팀청소기", "애완동물 털 제거기"],
    location: {
      type: "아파트",
      floor: "10층",
      area: "70㎡",
      parking: "지하주차장",
      elevator: true,
    },
    contact: {
      primary: "010-3333-4444",
      secondary: "010-9999-0000",
      emergency: "010-5555-6666",
    },
    schedule: {
      arrivalTime: "13:15",
      estimatedDuration: "2시간",
      bufferTime: "30분",
    },
    requirements: {
      specialAccess: "관리사무소 통과 필요",
      parkingInfo: "지하 1층 A구역",
      entryCode: "7890#",
    },
  },
  {
    id: 5,
    customer: "정민호",
    phone: "010-5555-6666",
    address: "서울 강서구 공항로 202",
    date: "2024-03-22",
    time: "10:00",
    partySize: 5,
    status: "pending",
    notes: "아침 일찍 방문해주세요.",
    serviceType: "이사청소",
    duration: "3시간",
    priority: "낮음",
    paymentStatus: "결제완료",
    totalAmount: 200000,
    startTime: null,
    completeTime: null,
    assignedWorker: "강민지",
    customerRating: 4.6,
    specialRequests: ["주방 청소", "욕실 소독"],
    equipment: ["진공청소기", "스팀청소기", "소독기"],
    location: {
      type: "오피스텔",
      floor: "12층",
      area: "60㎡",
      parking: "노상주차",
      elevator: true,
    },
    contact: {
      primary: "010-5555-6666",
      secondary: "010-7777-8888",
      emergency: "010-1111-2222",
    },
    schedule: {
      arrivalTime: "9:45",
      estimatedDuration: "3시간",
      bufferTime: "45분",
    },
    requirements: {
      specialAccess: "경비실 통과 필요",
      parkingInfo: "건물 앞 노상주차",
      entryCode: "3456#",
    },
  },
  {
    id: 6,
    customer: "한수진",
    phone: "010-7777-8888",
    address: "서울 종로구 종로 303",
    date: "2024-03-22",
    time: "15:00",
    partySize: 2,
    status: "pending",
    notes: "창문이 많아 창문 청소에 신경써주세요.",
    serviceType: "입주청소",
    duration: "4시간",
    priority: "높음",
    paymentStatus: "결제완료",
    totalAmount: 250000,
    startTime: null,
    completeTime: null,
    assignedWorker: "송준호",
    customerRating: 4.8,
    specialRequests: ["창문 청소", "카펫 청소"],
    equipment: ["진공청소기", "스팀청소기", "창문청소기"],
    location: {
      type: "아파트",
      floor: "18층",
      area: "90㎡",
      parking: "지하주차장",
      elevator: true,
    },
    contact: {
      primary: "010-7777-8888",
      secondary: "010-1111-2222",
      emergency: "010-9999-0000",
    },
    schedule: {
      arrivalTime: "14:45",
      estimatedDuration: "4시간",
      bufferTime: "1시간",
    },
    requirements: {
      specialAccess: "관리사무소 통과 필요",
      parkingInfo: "지하 1층 C구역",
      entryCode: "1234#",
    },
  },
  {
    id: 7,
    customer: "송지훈",
    phone: "010-9999-0000",
    address: "서울 용산구 이태원로 404",
    date: "2024-03-23",
    time: "11:30",
    partySize: 4,
    status: "pending",
    notes: "특별한 요청사항 없음",
    serviceType: "정기청소",
    duration: "2시간",
    priority: "보통",
    paymentStatus: "결제완료",
    totalAmount: 150000,
    startTime: null,
    completeTime: null,
    assignedWorker: "한지원",
    customerRating: 4.5,
    specialRequests: [],
    equipment: ["진공청소기", "스팀청소기"],
    location: {
      type: "아파트",
      floor: "5층",
      area: "50㎡",
      parking: "지하주차장",
      elevator: true,
    },
    contact: {
      primary: "010-9999-0000",
      secondary: "010-3333-4444",
      emergency: "010-5555-6666",
    },
    schedule: {
      arrivalTime: "11:15",
      estimatedDuration: "2시간",
      bufferTime: "30분",
    },
    requirements: {
      specialAccess: "관리사무소 통과 필요",
      parkingInfo: "지하 1층 D구역",
      entryCode: "5678#",
    },
  },
  {
    id: 8,
    customer: "강미영",
    phone: "010-2222-3333",
    address: "서울 성북구 안암로 505",
    date: "2024-03-23",
    time: "14:00",
    partySize: 3,
    status: "pending",
    notes: "알레르기가 있으니 먼지 제거에 신경써주세요.",
    serviceType: "이사청소",
    duration: "3시간",
    priority: "높음",
    paymentStatus: "결제완료",
    totalAmount: 200000,
    startTime: null,
    completeTime: null,
    assignedWorker: "임성민",
    customerRating: 4.9,
    specialRequests: ["먼지 제거", "공기청정기 필터 교체"],
    equipment: ["진공청소기", "스팀청소기", "공기청정기"],
    location: {
      type: "아파트",
      floor: "7층",
      area: "60㎡",
      parking: "지하주차장",
      elevator: true,
    },
    contact: {
      primary: "010-2222-3333",
      secondary: "010-8888-9999",
      emergency: "010-4444-5555",
    },
    schedule: {
      arrivalTime: "13:45",
      estimatedDuration: "3시간",
      bufferTime: "45분",
    },
    requirements: {
      specialAccess: "관리사무소 통과 필요",
      parkingInfo: "지하 1층 E구역",
      entryCode: "9012#",
    },
  },
  {
    id: 9,
    customer: "임동현",
    phone: "010-4444-5555",
    address: "서울 동작구 상도로 606",
    date: "2024-03-24",
    time: "10:30",
    partySize: 6,
    status: "pending",
    notes: "아침 일찍 방문해주세요.",
    serviceType: "입주청소",
    duration: "4시간",
    priority: "보통",
    paymentStatus: "결제완료",
    totalAmount: 250000,
    startTime: null,
    completeTime: null,
    assignedWorker: "조유진",
    customerRating: 4.7,
    specialRequests: ["전체 소독", "창문 청소"],
    equipment: ["진공청소기", "스팀청소기", "소독기", "창문청소기"],
    location: {
      type: "빌라",
      floor: "2층",
      area: "100㎡",
      parking: "전용주차장",
      elevator: false,
    },
    contact: {
      primary: "010-4444-5555",
      secondary: "010-6666-7777",
      emergency: "010-8888-9999",
    },
    schedule: {
      arrivalTime: "9:45",
      estimatedDuration: "4시간",
      bufferTime: "1시간",
    },
    requirements: {
      specialAccess: "현관 비밀번호 필요",
      parkingInfo: "전용주차장 2번",
      entryCode: "7890#",
    },
  },
  {
    id: 10,
    customer: "오서연",
    phone: "010-6666-7777",
    address: "서울 중구 을지로 707",
    date: "2024-03-24",
    time: "13:00",
    partySize: 2,
    status: "pending",
    notes: "특별한 요청사항 없음",
    serviceType: "정기청소",
    duration: "2시간",
    priority: "낮음",
    paymentStatus: "결제완료",
    totalAmount: 150000,
    startTime: null,
    completeTime: null,
    assignedWorker: "윤서연",
    customerRating: 4.6,
    specialRequests: [],
    equipment: ["진공청소기", "스팀청소기"],
    location: {
      type: "아파트",
      floor: "3층",
      area: "40㎡",
      parking: "지하주차장",
      elevator: true,
    },
    contact: {
      primary: "010-6666-7777",
      secondary: "010-2222-3333",
      emergency: "010-8888-9999",
    },
    schedule: {
      arrivalTime: "12:45",
      estimatedDuration: "2시간",
      bufferTime: "30분",
    },
    requirements: {
      specialAccess: "관리사무소 통과 필요",
      parkingInfo: "지하 1층 F구역",
      entryCode: "3456#",
    },
  },
  {
    id: 1,
    customer: "김철수",
    phone: "010-1234-5678",
    address: "서울 강남구 테헤란로 123",
    date: "2024-03-20",
    time: "14:00",
    partySize: 4,
    status: "pending",
    notes: "고객이 알레르기가 있으니 주의해주세요.",
    serviceType: "정기청소",
    duration: "2시간",
    priority: "높음",
    paymentStatus: "결제완료",
    totalAmount: 150000,
    startTime: null,
    completeTime: null,
    assignedWorker: "이지은",
    customerRating: 4.8,
    specialRequests: ["창문 청소", "카펫 청소"],
    equipment: ["진공청소기", "스팀청소기", "창문청소기"],
    location: {
      type: "아파트",
      floor: "15층",
      area: "84㎡",
      parking: "지하주차장",
      elevator: true,
    },
    contact: {
      primary: "010-1234-5678",
      secondary: "010-8765-4321",
      emergency: "010-1111-2222",
    },
    schedule: {
      arrivalTime: "13:45",
      estimatedDuration: "2시간",
      bufferTime: "30분",
    },
    requirements: {
      specialAccess: "관리사무소 통과 필요",
      parkingInfo: "지하 2층 B구역",
      entryCode: "1234#",
    },
  },
  {
    id: 2,
    customer: "이영희",
    phone: "010-8765-4321",
    address: "서울 마포구 서강로 456",
    date: "2024-03-20",
    time: "16:30",
    partySize: 2,
    status: "in_progress",
    startTime: "2024-03-20T16:30:00",
    notes: "주차 공간이 협소하니 주의해주세요.",
    serviceType: "이사청소",
    duration: "3시간",
    priority: "보통",
    paymentStatus: "결제완료",
    totalAmount: 200000,
    completeTime: null,
    assignedWorker: "최윤호",
    customerRating: 4.5,
    specialRequests: ["냉장고 청소", "욕실 소독"],
    equipment: ["진공청소기", "스팀청소기", "소독기"],
    location: {
      type: "오피스텔",
      floor: "8층",
      area: "45㎡",
      parking: "노상주차",
      elevator: true,
    },
    contact: {
      primary: "010-8765-4321",
      secondary: "010-1234-5678",
      emergency: "010-3333-4444",
    },
    schedule: {
      arrivalTime: "16:15",
      estimatedDuration: "3시간",
      bufferTime: "45분",
    },
    requirements: {
      specialAccess: "경비실 통과 필요",
      parkingInfo: "건물 앞 노상주차",
      entryCode: "5678#",
    },
  },
]);
// 필터기능
const filteredJobs = computed(() => {
  // assignedJobs.value 배열을 복사해서 result담는다
  let result = [...assignedJobs.value];
  //   검색기능 필터링
  if (searchQuery.value) {
    // 입력된 검색어를 소문자로 변환해서 query에 저장
    const query = searchQuery.value.toLowerCase();
    result = result.filter((job) => {
      //   console.log(job);
      return job.customer.toLowerCase().includes(query) || job.address.toLowerCase().includes(query);
    });
  }
  //   상태로 필터링
  if (stausFilter.value !== "all") {
    result = result.filter((job) => {
      return job.status === stausFilter.value;
    });
  }

  return result; // 최종 필터된 결과
});
// 페이지네이션
// 페이지네이션 관련 상태 추가
// 현재 페이지 번호
const currentPage = ref(1);
// 한페이지에 보여줄 항목수
const itemsPerPage = ref(5);
// 페이지네이션 계산
// 전체 페이지수 계산
const totalPages = computed(() => {
  return Math.ceil(filteredJobs.value.length / itemsPerPage.value); //2페이지
});
// 페이지네이션 적용
const paginatedJobs = computed(() => {
  // 현재 페이지에서 시작할 인덱스
  const start = (currentPage.value - 1) * itemsPerPage.value;
  // 끝 인덱스 (start + 한페이지에 표시할 수)
  const end = start + itemsPerPage.value;
  //   해당 범위만 잘라서 반환
  return filteredJobs.value.slice(start, end);
});
// 페이지에 클릭시
const goToPage = (page) => {
  if (page >= 1 && page <= totalPages.value) {
    currentPage.value = page;
  }
};
// 이전버튼
const prevPage = () => {
  if (currentPage.value > 1) {
    currentPage.value--;
  }
};
// 다음버튼
const nextPage = () => {
  if (currentPage.value < totalPages.value) {
    currentPage.value++; //현재페이지 번호 증가
  }
};
// // 한글이슈
function handleInput(event) {
  searchQuery.value = event.target.value;
}
// 상태 글자 변환
const getStatusText = (status) => {
  const statusMap = {
    pending: "대기중",
    in_progress: "진행중",
    completed: "완료",
  };
  return statusMap[status] || status;
};
// 상태css적용
const getStatusClass = (status) => {
  const statusClasses = {
    pending: "bg-yellow-100 text-yellow-800",
    in_progress: "bg-blue-100 text-blue-800",
    completed: "bg-green-100 text-green-800",
  };
  return statusClasses[status] || "bg-gray-100 text-gray-800";
};
// 상태 아이콘

const getStatusIcon = (status) => {
  const statusIcons = {
    pending: "fas fa-clock text-yellow-500",
    in_progress: "fas fa-spinner fa-spin text-blue-500",
    completed: "fas fa-check-circle text-green-500",
  };
  return statusIcons[status] || "fas fa-circle text-gray-500";
};
const formatDate = (date) => {
  return new Date(date).toLocaleDateString("ko-KR", {
    year: "numeric",
    month: "long",
    day: "numeric",
    weekday: "long",
  });
};
// 시작 클릭시
const startJob = (job) => {
  // 전달받은 job의 id와 같은 id를 가진 작업이 assignedJobs배열에서 몇번째에 있는지 찾음
  const index = assignedJobs.value.findIndex((j) => j.id === job.id);
  // 같은 id를 가진 작업이 배열에 실제 로 존재하는 경우
  if (index !== -1) {
    assignedJobs.value[index] = {
      ...job,
      status: "in_progress",
      startTime: new Date().toISOString(), // 시작 시간 기록
    };
  }
};
// 완료 클릭시
const completeJob = (job) => {
  const index = assignedJobs.value.findIndex((j) => j.id === job.id);
  if (index !== -1) {
    assignedJobs.value[index] = {
      ...job,
      status: "completed",
      completeTime: new Date().toISOString(), // 시작 시간 기록
    };
  }
  //   상세정보에 완료버튼을 누르면 모달닫기
  //   사용자가 이미선택한 작업을 다시 클릭한 경우
  if (selectedJob.value && selectedJob.value.id === job.id) {
    closeModal(); // 모달을 닫아서 선택을 취소하거나 중복 선택을 방지
  }
};
// 상세모달
// null 개발자가 직접설정 비어있음 (아직 선택 안된상태)
const selectedJob = ref(null);
const showJobDetails = (job) => {
  selectedJob.value = { ...job };
};
// 숫자 금액을 한국 통화 형식(₩)으로 변환하는 함수
const formatCurrency = (amount) => {
  return new Intl.NumberFormat("ko-KR", {
    style: "currency", // 통화 형식 지정
    currency: "KRW", // 대한민국 원화
    maximumFractionDigits: 0, // 소수점 자릿수 0자리 (₩12,000 처럼 표시)
  }).format(amount); // 실제 금액을 포맷팅해서 반환
};
const formatTime = (time) => {
  return new Date(time).toLocaleTimeString("ko-KR", {
    hour: "2-digit",
    minute: "2-digit",
  });
};
// 닫기 버튼
const closeModal = () => {
  selectedJob.value = null;
};
</script>
