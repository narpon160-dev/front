<template>
  <div class="home-container">
    <!-- รูปภาพพื้นหลังแบบเบลอ (แก้ความโล่ง) -->
    <div class="blurred-background"></div>
    <div class="background-overlay"></div>

    <!-- Navbar -->
    <header class="navbar">
      <div class="brand">
        <span class="icon">🏢</span>
        อำพัน บ้านเช่า
      </div>
      <button class="btn-portal" @click="isSelectionPopupVisible = true">
        🧾 เลือกห้องพัก
      </button>
    </header>

    <!-- เนื้อหาหลัก -->
    <main class="hero-section">
      <!-- กล่องข้อความด้านซ้าย -->
      <div class="hero-content-card">
        <h1 class="hero-title">
          ยินดีต้อนรับสู่<br />
          <span class="highlight">อำพัน บ้านเช่า</span>
        </h1>
        <p class="hero-subtitle">
          สัมผัสประสบการณ์การพักผ่อนที่เหนือกว่า ด้วยห้องพักที่สะอาด ปลอดภัย
          ตกแต่งสไตล์โมเดิร์น พร้อมสิ่งอำนวยความสะดวกครบครัน
          และระบบจัดการบิลอัตโนมัติ
        </p>

        <!-- กล่องคุณสมบัติ 3 สี -->
        <div class="features-grid">
          <div class="feature-card bg-green">
            <div class="feature-icon">🌿</div>
            <div class="feature-text">บรรยากาศ<br />ร่มรื่น</div>
          </div>
          <div class="feature-card bg-blue">
            <div class="feature-icon">🔒</div>
            <div class="feature-text">ความปลอดภัย<br />24 ชม.</div>
          </div>
          <div class="feature-card bg-pink">
            <div class="feature-icon">📍</div>
            <div class="feature-text">เดินทาง<br />สะดวก</div>
          </div>
        </div>
      </div>

      <!-- กล่องรูปภาพด้านขวา -->
      <div class="hero-image-wrapper">
        <div class="image-decorative-bg"></div>
        <img
          src="https://images.unsplash.com/photo-1522708323590-d24dbb6b0267?ixlib=rb-4.0.3&auto=format&fit=crop&w=800&q=80"
          alt="รูปห้องพัก อำพัน บ้านเช่า"
          class="hero-main-image"
        />
      </div>
    </main>

    <!-- Footer ปิดท้าย -->
    <footer class="simple-footer">
      <p>
        📍 อำพัน บ้านเช่า | บริการห้องพักรายเดือนคุณภาพ สาขาชากค้อ และ บางละมุง
      </p>
      <p class="footer-sub">© 2026 Ampun House. All rights reserved.</p>
    </footer>

    <!-- ==============================================
         ป๊อปอัปเลือกบ้านและห้อง
         ============================================== -->
    <div
      v-if="isSelectionPopupVisible"
      class="modal-overlay"
      @click.self="isSelectionPopupVisible = false"
    >
      <div class="modal-content">
        <div class="modal-header">
          <h3>🧾 ระบุสถานที่และเลขห้อง</h3>
          <button class="btn-close" @click="isSelectionPopupVisible = false">
            ✖
          </button>
        </div>

        <div class="modal-body receipt-slip">
          <div class="location-selection">
            <h4 class="step-title">1. เลือกสถานที่</h4>
            <div class="location-cards">
              <div
                v-for="(rooms, locName) in locationsData"
                :key="locName"
                class="location-card"
                :class="{ selected: selectedLocation === locName }"
                @click="selectLocation(locName)"
              >
                <div class="location-icon">
                  <span v-if="locName === 'ชากค้อ'">🏢🏭</span>
                  <span v-else>🏖️⛵</span>
                </div>
                <div class="location-info">
                  <p class="loc-name">{{ locName }}</p>
                  <p class="loc-desc" v-if="locName === 'ชากค้อ'">
                    โมเดิร์นและทันสมัย ใกล้แหล่งงาน
                  </p>
                  <p class="loc-desc" v-else>
                    วิถีชีวิตชายฝั่ง บรรยากาศเงียบสงบ
                  </p>
                </div>
              </div>
            </div>
          </div>

          <div v-if="selectedLocation" class="room-selection">
            <h4 class="step-title">2. เลือกเลขห้อง</h4>
            <div class="room-grid">
              <button
                v-for="room in locationsData[selectedLocation]"
                :key="room"
                class="room-btn"
                :class="{ selected: selectedRoom === room }"
                @click="selectedRoom = room"
              >
                {{ room }}
              </button>
            </div>
          </div>

          <div class="receipt-footer">
            เมื่อเลือกห้องแล้ว กดปุ่มยืนยันเพื่อเข้าสู่ระบบบิล
          </div>
        </div>

        <div class="modal-footer">
          <button
            class="btn-confirm"
            :disabled="!selectedRoom"
            @click="confirmAndProceed"
          >
            ยืนยันและดำเนินการต่อ
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";

const emit = defineEmits(["go-to-cal"]);

const isSelectionPopupVisible = ref(false);
const selectedLocation = ref(null);
const selectedRoom = ref(null);

const locationsData = ref({
  ชากค้อ: [
    "1",
    "2",
    "3",
    "4",
    "5",
    "6",
    "7",
    "8",
    "9",
    "10",
    "101",
    "102",
    "103",
    "104",
    "105",
    "201",
    "202",
    "203",
    "204",
    "205",
    "206",
  ],
  บางละมุง: [
    "1",
    "2",
    "3",
    "4",
    "5",
    "บ้านบุ่ย",
    "บ้านลูกเเก้ว",
    "โอ๋",
    "บ้านสีฟ้า",
    "เบ้น",
  ],
});

const selectLocation = (locationName) => {
  selectedLocation.value = locationName;
  selectedRoom.value = null;
};

const confirmAndProceed = () => {
  if (selectedLocation.value && selectedRoom.value) {
    emit("go-to-cal", {
      location: selectedLocation.value,
      room: selectedRoom.value,
    });
    isSelectionPopupVisible.value = false;
  } else {
    alert("กรุณาเลือกสถานที่และเลขห้องพักให้ครบถ้วนก่อนดำเนินการต่อ");
  }
};
</script>

<style scoped>
/* โครงสร้างหลัก */
.home-container {
  min-height: 100vh;
  background: transparent;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  display: flex;
  flex-direction: column;
  position: relative;
  overflow-x: hidden;
  overflow-y: auto;
}

/* =========================================
   รูปภาพพื้นหลังเบลอ (Blurred Background)
   ========================================= */
.blurred-background {
  position: fixed;
  top: -20px;
  left: -20px;
  right: -20px;
  bottom: -20px;
  background-image: url("https://images.unsplash.com/photo-1522771731535-6fedf123286b?ixlib=rb-4.0.3&auto=format&fit=crop&w=1920&q=80");
  background-size: cover;
  background-position: center;
  filter: blur(12px);
  z-index: -2;
}

.background-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(255, 255, 255, 0.75);
  z-index: -1;
}

/* =========================================
   ดันเนื้อหาทั้งหมดให้ลอยอยู่เหนือพื้นหลัง
   ========================================= */
.navbar,
.hero-section,
.simple-footer {
  position: relative;
  z-index: 10;
}

/* แถบนำทาง (Navbar) */
.navbar {
  background: rgba(255, 255, 255, 0.85);
  backdrop-filter: blur(10px);
  padding: 16px 50px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  position: sticky;
  top: 0;
  border-bottom: 1px solid rgba(255, 255, 255, 0.5);
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.05);
}
.brand {
  font-size: 24px;
  font-weight: 800;
  color: #1e293b;
  display: flex;
  align-items: center;
  gap: 12px;
}
.btn-portal {
  background-color: #475569;
  color: white;
  padding: 12px 28px;
  border-radius: 50px;
  border: none;
  font-size: 15px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
}
.btn-portal:hover {
  background-color: #334155;
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(71, 85, 105, 0.3);
}

/* ส่วนเนื้อหาหลัก (Hero Section) */
.hero-section {
  max-width: 1300px;
  margin: auto;
  padding: 40px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 60px;
  flex: 1;
}

.hero-content-card {
  flex: 1;
  background: rgba(255, 255, 255, 0.95);
  padding: 50px;
  border-radius: 32px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.08);
  max-width: 600px;
  border: 1px solid #ffffff;
}
.hero-title {
  font-size: 46px;
  line-height: 1.3;
  font-weight: 800;
  color: #0f172a;
  margin-top: 0;
  margin-bottom: 20px;
}
.highlight {
  color: #0284c7;
}
.hero-subtitle {
  font-size: 17px;
  line-height: 1.7;
  color: #475569;
  margin-bottom: 40px;
}

/* กล่องคุณสมบัติ */
.features-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
}
.feature-card {
  padding: 25px 15px;
  border-radius: 20px;
  text-align: center;
  transition: transform 0.3s ease;
}
.feature-card:hover {
  transform: translateY(-5px);
}
.feature-icon {
  font-size: 32px;
  margin-bottom: 12px;
}
.feature-text {
  font-size: 14px;
  font-weight: 700;
  line-height: 1.4;
}
.bg-green {
  background-color: #d1fae5;
  color: #065f46;
  border: 1px solid #bbf7d0;
}
.bg-blue {
  background-color: #e0f2fe;
  color: #075985;
  border: 1px solid #bae6fd;
}
.bg-pink {
  background-color: #ffe4e6;
  color: #9f1239;
  border: 1px solid #fecdd3;
}

/* กล่องรูปภาพด้านขวา */
.hero-image-wrapper {
  flex: 1;
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
}
.image-decorative-bg {
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  background: linear-gradient(135deg, #0ea5e9 0%, #38bdf8 50%, #bae6fd 100%);
  border-radius: 40px;
  transform: rotate(3deg);
  z-index: 1;
  opacity: 0.2;
}
.hero-main-image {
  width: 100%;
  max-width: 550px;
  aspect-ratio: 4/3;
  object-fit: cover;
  border-radius: 30px;
  position: relative;
  z-index: 2;
  box-shadow: 0 30px 60px rgba(0, 0, 0, 0.15);
  border: 8px solid #ffffff;
}

/* แถบปิดท้าย (Footer) */
.simple-footer {
  text-align: center;
  padding: 25px 20px;
  background: rgba(255, 255, 255, 0.85);
  backdrop-filter: blur(10px);
  border-top: 1px solid rgba(255, 255, 255, 0.5);
}
.simple-footer p {
  margin: 0 0 5px;
  color: #475569;
  font-size: 14.5px;
  font-weight: 600;
}
.simple-footer .footer-sub {
  font-size: 13px;
  color: #94a3b8;
  font-weight: 400;
}

/* =========================================
   สไตล์ Modal (ส่วนเลือกบ้าน/ห้อง) ป้องกันทะลุจอ
   ========================================= */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%; /* เปลี่ยนจาก 100vw เป็น 100% */
  height: 100%; /* เปลี่ยนจาก 100vh เป็น 100% */
  background-color: rgba(15, 23, 42, 0.6);
  backdrop-filter: blur(5px);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  padding: 20px;
  box-sizing: border-box; /* เพิ่มเพื่อป้องกัน padding ดันทะลุจอ */
}
.modal-content {
  background-color: white;
  border-radius: 16px;
  width: 100%;
  max-width: 550px;
  max-height: 90vh; /* จำกัดความสูงไม่ให้ล้นแนวตั้ง */
  box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.25);
  overflow: hidden;
  display: flex;
  flex-direction: column;
  animation: modalIn 0.3s ease-out;
  box-sizing: border-box; /* เพิ่มเพื่อป้องกันทะลุ */
}
@keyframes modalIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
.modal-header {
  background-color: #f8fafc;
  padding: 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid #e2e8f0;
}
.modal-header h3 {
  margin: 0;
  font-size: 18px;
  color: #0f172a;
  font-weight: 700;
}
.btn-close {
  background: none;
  border: none;
  font-size: 20px;
  cursor: pointer;
  color: #94a3b8;
  transition: 0.2s;
}
.btn-close:hover {
  color: #ef4444;
}
.modal-body {
  padding: 30px;
  flex: 1;
  overflow-y: auto;
}
.step-title {
  margin-top: 0;
  margin-bottom: 15px;
  font-size: 16px;
  color: #475569;
  font-weight: 700;
}
.location-cards {
  display: flex;
  flex-direction: column;
  gap: 15px;
  margin-bottom: 30px;
}
.location-card {
  background-color: #f8fafc;
  border: 2px solid #e2e8f0;
  border-radius: 12px;
  padding: 15px 20px;
  display: flex;
  align-items: center;
  gap: 15px;
  cursor: pointer;
  transition: all 0.2s;
}
.location-card:hover {
  background-color: #f0f9ff;
  border-color: #bae6fd;
}
.location-card.selected {
  background-color: #e0f2fe;
  border-color: #0284c7;
  box-shadow: 0 4px 12px rgba(2, 132, 199, 0.15);
}
.location-icon {
  font-size: 28px;
}
.loc-name {
  margin: 0 0 4px;
  font-size: 16px;
  color: #1e293b;
  font-weight: 700;
}
.loc-desc {
  margin: 0;
  font-size: 13.5px;
  color: #64748b;
}
.room-selection {
  border-top: 2px dashed #e2e8f0;
  padding-top: 25px;
}
.room-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(65px, 1fr));
  gap: 12px;
}
.room-btn {
  background-color: #ffffff;
  border: 1px solid #cbd5e1;
  color: #475569;
  font-weight: 600;
  padding: 12px 10px;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s;
  text-align: center;
  font-size: 15px;
}
.room-btn:hover {
  background-color: #f0f9ff;
  border-color: #7dd3fc;
  color: #0284c7;
  transform: translateY(-2px);
}
.room-btn.selected {
  background-color: #0284c7;
  color: white;
  border-color: #0284c7;
  box-shadow: 0 4px 10px rgba(2, 132, 199, 0.3);
  transform: translateY(-2px);
}
.receipt-footer {
  text-align: center;
  font-size: 13px;
  color: #94a3b8;
  margin-top: 25px;
}
.modal-footer {
  padding: 20px;
  background-color: #f8fafc;
  border-top: 1px solid #e2e8f0;
  display: flex;
  justify-content: flex-end;
}
.btn-confirm {
  background-color: #0284c7;
  color: white;
  padding: 12px 24px;
  border-radius: 8px;
  border: none;
  font-size: 15px;
  font-weight: 700;
  cursor: pointer;
  transition: 0.2s;
}
.btn-confirm:hover:not(:disabled) {
  background-color: #0369a1;
}
.btn-confirm:disabled {
  background-color: #cbd5e1;
  color: #ffffff;
  cursor: not-allowed;
}

/* Responsive Design สำหรับมือถือ */
@media (max-width: 968px) {
  .hero-section {
    flex-direction: column;
    gap: 40px;
    padding: 20px;
    margin: 30px auto;
  }
  .hero-content-card {
    padding: 30px 20px;
    text-align: center;
  }
  .hero-title {
    font-size: 32px;
  }
  .features-grid {
    gap: 10px;
  }
  .feature-card {
    padding: 15px 10px;
  }
  .feature-icon {
    font-size: 24px;
  }
  .feature-text {
    font-size: 12px;
  }
  .hero-main-image {
    max-width: 100%;
    border-width: 4px;
  }
  .image-decorative-bg {
    border-radius: 20px;
  }
  .navbar {
    padding: 15px 20px;
  }
  .brand {
    font-size: 20px;
  }

  /* เพิ่มส่วนจัดการ Modal บนมือถือ */
  .modal-overlay {
    padding: 15px;
  }
  .modal-body {
    padding: 20px;
  }
  .modal-footer {
    padding: 15px;
  }
  .btn-confirm {
    width: 100%; /* ให้ปุ่มกดยืนยันเต็มจอ กดง่ายขึ้น */
    padding: 14px;
    font-size: 16px;
  }
}
</style>
