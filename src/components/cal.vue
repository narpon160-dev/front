<template>
  <div class="bill-container">
    <div class="header-actions hide-on-print">
      <button class="btn-back" @click="$emit('go-back')">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="16"
          height="16"
          fill="currentColor"
          viewBox="0 0 16 16"
        >
          <path
            fill-rule="evenodd"
            d="M15 8a.5.5 0 0 0-.5-.5H2.707l3.147-3.146a.5.5 0 1 0-.708-.708l-4 4a.5.5 0 0 0 0 .708l4 4a.5.5 0 0 0 .708-.708L2.707 8.5H14.5A.5.5 0 0 0 15 8z"
          />
        </svg>
        กลับหน้าหลัก
      </button>
    </div>

    <!-- เปลี่ยนชื่อหัวข้อตามสถานะ (สร้างใหม่ / แก้ไข) -->
    <h2 class="page-title hide-on-print">
      {{ isEditing ? "✏️ แก้ไขใบบิลค่าเช่า" : "🧾 ใบบิลค่าเช่า" }}
    </h2>

    <div class="info-row hide-on-print">
      <div class="input-group">
        <label>ห้องที่:</label>
        <input
          v-model="roomNumber"
          type="text"
          placeholder="เช่น A101"
          readonly
          style="background-color: #f1f5f9 !important"
        />
      </div>
      <div class="input-group">
        <label>วันที่:</label>
        <input v-model="billDate" type="date" />
      </div>
    </div>

    <div class="table-wrapper hide-on-print">
      <table class="responsive-table">
        <thead>
          <tr>
            <th>รายการ</th>
            <th>เลขครั้งที่</th>
            <th>เลขครั้งก่อน</th>
            <th>หน่วยใช้ไป</th>
            <th>หน่วยละ</th>
            <th>จำนวนเงิน (บาท)</th>
          </tr>
        </thead>
        <tbody>
          <!-- ค่าไฟ -->
          <tr>
            <td data-label="รายการ" class="item-name">⚡ ค่าไฟฟ้า</td>
            <td data-label="เลขครั้งที่">
              <input type="number" v-model.number="electricity.current" />
            </td>
            <td data-label="เลขครั้งก่อน">
              <input type="number" v-model.number="electricity.previous" />
            </td>
            <td data-label="หน่วยใช้ไป" class="unit-val">
              {{ electricityUnits }}
            </td>
            <td data-label="หน่วยละ">
              <input type="number" v-model.number="electricity.rate" />
            </td>
            <td data-label="จำนวนเงิน" class="cost-val">
              {{ electricityCost.toLocaleString() }}
            </td>
          </tr>
          <!-- ค่าน้ำ -->
          <tr>
            <td data-label="รายการ" class="item-name">💧 ค่าน้ำประปา</td>
            <td data-label="เลขครั้งที่">
              <input type="number" v-model.number="water.current" />
            </td>
            <td data-label="เลขครั้งก่อน">
              <input type="number" v-model.number="water.previous" />
            </td>
            <td data-label="หน่วยใช้ไป" class="unit-val">{{ waterUnits }}</td>
            <td data-label="หน่วยละ">
              <input type="number" v-model.number="water.rate" />
            </td>
            <td data-label="จำนวนเงิน" class="cost-val">
              {{ waterCost.toLocaleString() }}
            </td>
          </tr>
          <!-- ค่าเช่า -->
          <tr class="highlight-row">
            <td data-label="รายการ" class="item-name">🏠 ค่าเช่าห้อง</td>
            <td class="hide-on-mobile" colspan="4"></td>
            <td data-label="ค่าเช่า (บาท)">
              <input
                type="number"
                v-model.number="roomRent"
                class="rent-input"
              />
            </td>
          </tr>

          <!-- รายการอื่นๆ ที่เพิ่มเข้ามาแบบไดนามิก -->
          <tr
            v-for="(extra, index) in extraItems"
            :key="index"
            class="highlight-row"
          >
            <td data-label="รายการ">
              <div class="extra-item-name">
                <button
                  class="btn-delete-sm"
                  @click="removeExtraItem(index)"
                  title="ลบรายการนี้"
                >
                  ✖
                </button>
                <input
                  type="text"
                  v-model="extra.name"
                  placeholder="ระบุชื่อรายการ (เช่น ค่าขยะ)"
                />
              </div>
            </td>
            <td class="hide-on-mobile" colspan="4"></td>
            <td data-label="จำนวนเงิน (บาท)">
              <input type="number" v-model.number="extra.cost" />
            </td>
          </tr>

          <!-- ปุ่มเพิ่มรายการ -->
          <tr class="add-item-row">
            <td colspan="6">
              <button class="btn-add-item" @click="addExtraItem">
                + เพิ่มรายการอื่นๆ
              </button>
            </td>
          </tr>

          <tr class="total-row">
            <td
              data-label="รวมเป็นเงินสุทธิ"
              class="total-label hide-on-mobile"
              colspan="5"
            >
              รวมเป็นเงินสุทธิ
            </td>
            <td data-label="ยอดชำระสุทธิ" class="total-amount">
              {{ totalCost.toLocaleString() }} <span class="currency">บาท</span>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- เปลี่ยนปุ่มตามสถานะ -->
    <div class="action-buttons-group hide-on-print">
      <button v-if="!isEditing" class="btn-save" @click="saveBill">
        💾 บันทึกข้อมูลลงระบบ
      </button>
      <div v-else class="edit-buttons">
        <button class="btn-update" @click="saveBill">🔄 อัปเดตข้อมูล</button>
        <button class="btn-cancel-edit" @click="cancelEdit">
          ❌ ยกเลิกแก้ไข
        </button>
      </div>
    </div>

    <div class="remark hide-on-print">
      <strong>📌 หมายเหตุ:</strong>
      <ul>
        <li>
          เก็บเงินทุกวันที่ 2 ของเดือน กรณีไม่อยู่ให้ฝากไว้ที่ร้านค้าป้าสงวน ตึก
          A ห้อง 5
        </li>
        <li>
          จ่ายค่าเช่าไม่เกินทุกวันที่ 5 ของเดือน ถ้าเกินคิดค่าปรับวันละ 100 บาท
          ไม่ว่ากรณีใดๆ ทั้งสิ้น
        </li>
      </ul>
    </div>

    <div class="history-section hide-on-print">
      <h3>📋 ประวัติบิลสำหรับห้อง: {{ roomNumber }}</h3>
      <div v-if="filteredBillsHistory.length > 0" class="history-grid">
        <div
          class="history-card"
          v-for="item in filteredBillsHistory"
          :key="item._id"
        >
          <div class="history-header">
            <span class="room-highlight">{{ item.roomNumber }}</span>
            <span class="history-date">{{ item.billDate }}</span>
          </div>
          <div class="history-body">
            <div class="cost-val">
              ยอด: {{ item.totalCost.toLocaleString() }} ฿
            </div>
            <div class="action-buttons">
              <button class="btn-view" @click="viewBill(item)">
                📄 พิมพ์บิล
              </button>
              <button class="btn-edit" @click="editBill(item)">✏️ แก้ไข</button>
              <button class="btn-delete" @click="deleteBill(item._id)">
                ลบ
              </button>
            </div>
          </div>
        </div>
      </div>
      <div v-else class="empty-state">
        ยังไม่มีประวัติการออกบิลสำหรับห้องนี้
      </div>
    </div>

    <!-- Popup ใบเสร็จ -->
    <div v-if="selectedBill" class="modal-overlay" @click.self="closeBill">
      <div class="custom-print-date">{{ printTimestamp }}</div>

      <div class="modal-content receipt-modal-content">
        <div class="modal-header hide-on-print">
          <h3>🧾 รายละเอียดบิลค่าเช่า</h3>
          <button class="btn-close" @click="closeBill">✖</button>
        </div>

        <div class="modal-body receipt-slip">
          <div class="receipt-header">
            <p>
              <strong>ห้องพัก:</strong>
              <span class="highlight-text">{{ selectedBill.roomNumber }}</span>
            </p>
            <p><strong>ประจำวันที่:</strong> {{ selectedBill.billDate }}</p>
          </div>

          <table class="receipt-table receipt-fixed-table">
            <thead>
              <tr>
                <th>รายการ</th>
                <th style="text-align: center">หน่วย</th>
                <th style="text-align: center">หน่วยละ (บาท)</th>
                <th style="text-align: right">จำนวนเงิน (บาท)</th>
              </tr>
            </thead>
            <tbody>
              <tr>
                <td>
                  <strong>ค่าไฟฟ้า</strong>
                  <div class="meter-detail">
                    (จดครั้งนี้: {{ selectedBill.electricity.current }} -
                    ครั้งก่อน: {{ selectedBill.electricity.previous }})
                  </div>
                </td>
                <td align="center">{{ selectedBill.electricity.units }}</td>
                <td align="center">
                  {{ selectedBill.electricity.rate.toLocaleString() }}
                </td>
                <td align="right">
                  {{ selectedBill.electricity.cost.toLocaleString() }}
                </td>
              </tr>
              <tr>
                <td>
                  <strong>ค่าน้ำประปา</strong>
                  <div class="meter-detail">
                    (จดครั้งนี้: {{ selectedBill.water.current }} - ครั้งก่อน:
                    {{ selectedBill.water.previous }})
                  </div>
                </td>
                <td align="center">{{ selectedBill.water.units }}</td>
                <td align="center">
                  {{ selectedBill.water.rate.toLocaleString() }}
                </td>
                <td align="right">
                  {{ selectedBill.water.cost.toLocaleString() }}
                </td>
              </tr>
              <tr>
                <td><strong>ค่าเช่าห้อง</strong></td>
                <td align="center">-</td>
                <td align="center">-</td>
                <td align="right">
                  {{ selectedBill.roomRent.toLocaleString() }}
                </td>
              </tr>

              <!-- ลูปโชว์รายการอื่นๆ ในใบเสร็จ -->
              <tr v-for="(extra, i) in selectedBill.extraItems" :key="i">
                <td>
                  <strong>{{ extra.name || "รายการอื่นๆ" }}</strong>
                </td>
                <td align="center">-</td>
                <td align="center">-</td>
                <td align="right">{{ extra.cost.toLocaleString() }}</td>
              </tr>
            </tbody>
            <tfoot>
              <tr>
                <td colspan="3" class="total-title">รวมเป็นเงินสุทธิ</td>
                <td class="total-price">
                  {{ selectedBill.totalCost.toLocaleString() }} ฿
                </td>
              </tr>
            </tfoot>
          </table>

          <div class="receipt-footer">
            * กรุณาชำระเงินภายในวันที่ 5 ของเดือน
          </div>
        </div>

        <div class="modal-footer hide-on-print">
          <button class="btn-print" @click="printSlip">🖨️ พิมพ์ใบเสร็จ</button>
          <button class="btn-cancel" @click="closeBill">ปิดหน้าต่าง</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from "vue";

const props = defineProps({
  roomData: { type: Object, default: null },
});

defineEmits(["go-back"]);

const formatRoomName = () => {
  if (props.roomData && props.roomData.location && props.roomData.room) {
    return `${props.roomData.location} - ห้อง ${props.roomData.room}`;
  }
  return "";
};

const getDefaultRent = () => {
  if (!props.roomData) return 3000;
  const loc = props.roomData.location;
  const room = props.roomData.room;

  if (loc === "ชากค้อ") {
    return 2000;
  } else if (loc === "บางละมุง") {
    if (room === "1" || room === "2") return 2700;
    else if (room === "3") return 3000;
    else if (room === "4" || room === "5") return 2500;
    else if (room === "เบ้น") return 3000;
    else return 4000;
  }
  return 3000;
};

const roomNumber = ref(formatRoomName());
const billDate = ref(new Date().toISOString().split("T")[0]);
const roomRent = ref(getDefaultRent());

const electricity = ref({ current: 0, previous: 0, rate: 7 });
const water = ref({ current: 0, previous: 0, rate: 25 });
const extraItems = ref([]);

const billsHistory = ref([]);
const selectedBill = ref(null);
const printTimestamp = ref("");

const isEditing = ref(false);
const editingId = ref(null);

const electricityUnits = computed(() =>
  Math.max(0, electricity.value.current - electricity.value.previous),
);
const waterUnits = computed(() =>
  Math.max(0, water.value.current - water.value.previous),
);
const electricityCost = computed(
  () => electricityUnits.value * electricity.value.rate,
);
const waterCost = computed(() => waterUnits.value * water.value.rate);
const extraItemsTotal = computed(() =>
  extraItems.value.reduce((sum, item) => sum + (item.cost || 0), 0),
);
const totalCost = computed(
  () =>
    electricityCost.value +
    waterCost.value +
    (roomRent.value || 0) +
    extraItemsTotal.value,
);

const filteredBillsHistory = computed(() => {
  if (!roomNumber.value) return billsHistory.value;
  return billsHistory.value.filter(
    (bill) => bill.roomNumber === roomNumber.value,
  );
});

const addExtraItem = () => {
  extraItems.value.push({ name: "", cost: 0 });
};
const removeExtraItem = (index) => {
  extraItems.value.splice(index, 1);
};

const loadSavedExtraItemsTemplate = () => {
  const saved = localStorage.getItem(`extraTemplate_${roomNumber.value}`);
  if (saved) {
    extraItems.value = JSON.parse(saved);
  } else {
    extraItems.value = [];
  }
};

watch(
  extraItems,
  (newVal) => {
    if (!isEditing.value) {
      localStorage.setItem(
        `extraTemplate_${roomNumber.value}`,
        JSON.stringify(newVal),
      );
    }
  },
  { deep: true },
);

const fetchBills = async () => {
  try {
    const res = await fetch("https://kkk-sf90.onrender.com/api/bills");
    billsHistory.value = await res.json();
  } catch (err) {
    console.error("ไม่สามารถดึงข้อมูลได้", err);
  }
};

const saveBill = async () => {
  if (!roomNumber.value) return alert("กรุณากรอกเลขห้องก่อน");

  const payload = {
    roomNumber: roomNumber.value,
    billDate: billDate.value,
    roomRent: roomRent.value,
    electricity: {
      ...electricity.value,
      units: electricityUnits.value,
      cost: electricityCost.value,
    },
    water: { ...water.value, units: waterUnits.value, cost: waterCost.value },
    extraItems: extraItems.value,
    totalCost: totalCost.value,
  };

  try {
    const url = isEditing.value
      ? `https://kkk-sf90.onrender.com/api/bills/${editingId.value}`
      : "https://kkk-sf90.onrender.com/api/bills";
    const method = isEditing.value ? "PUT" : "POST";

    const res = await fetch(url, {
      method: method,
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(payload),
    });

    if (res.ok) {
      alert(
        isEditing.value ? "✅ อัปเดตข้อมูลสำเร็จ!" : "✅ บันทึกบิลใหม่สำเร็จ!",
      );
      fetchBills();
      if (isEditing.value) cancelEdit();
    } else {
      alert("❌ บันทึกล้มเหลว");
    }
  } catch (err) {
    alert("❌ เชื่อมต่อฐานข้อมูลล้มเหลว");
  }
};

const editBill = (bill) => {
  isEditing.value = true;
  editingId.value = bill._id;
  billDate.value = bill.billDate;
  roomRent.value = bill.roomRent;
  electricity.value = {
    current: bill.electricity.current,
    previous: bill.electricity.previous,
    rate: bill.electricity.rate,
  };
  water.value = {
    current: bill.water.current,
    previous: bill.water.previous,
    rate: bill.water.rate,
  };
  extraItems.value = bill.extraItems ? [...bill.extraItems] : [];
  window.scrollTo({ top: 0, behavior: "smooth" });
};

const cancelEdit = () => {
  isEditing.value = false;
  editingId.value = null;
  billDate.value = new Date().toISOString().split("T")[0];
  roomRent.value = getDefaultRent();
  electricity.value = { current: 0, previous: 0, rate: 7 };
  water.value = { current: 0, previous: 0, rate: 25 };
  loadSavedExtraItemsTemplate();
};

const deleteBill = async (id) => {
  if (!confirm("ยืนยันลบบิลนี้? (ไม่สามารถกู้คืนได้)")) return;
  try {
    await fetch(`https://kkk-sf90.onrender.com/api/bills/${id}`, {
      method: "DELETE",
    });
    fetchBills();
  } catch (err) {
    alert("ลบไม่สำเร็จ");
  }
};

const viewBill = (bill) => {
  selectedBill.value = bill;
};
const closeBill = () => {
  selectedBill.value = null;
};

const printSlip = () => {
  const now = new Date();
  printTimestamp.value = now.toLocaleString("th-TH", {
    year: "numeric",
    month: "short",
    day: "numeric",
    hour: "2-digit",
    minute: "2-digit",
  });
  setTimeout(() => window.print(), 100);
};

onMounted(() => {
  fetchBills();
  loadSavedExtraItemsTemplate();
});
</script>

<style scoped>
.bill-container {
  max-width: 900px;
  margin: 20px auto;
  padding: 25px;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
  background: #ffffff;
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
  overflow-x: hidden;
}
.header-actions {
  margin-bottom: 20px;
}
.btn-back {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  background: #f1f5f9;
  border: none;
  color: #475569;
  font-size: 14px;
  padding: 8px 16px;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.2s;
}
.btn-back:hover {
  background: #e2e8f0;
  color: #0f172a;
}
.page-title {
  text-align: center;
  color: #0f172a;
  font-size: 24px;
  margin-bottom: 25px;
  font-weight: 800;
}
.info-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
  margin-bottom: 25px;
  background: #f8fafc;
  padding: 20px;
  border-radius: 10px;
  border: 1px solid #e2e8f0;
}
.input-group {
  display: flex;
  flex-direction: column;
  gap: 8px;
}
.input-group label {
  font-weight: 600;
  color: #475569;
  font-size: 14px;
}
input[type="number"],
input[type="text"],
input[type="date"] {
  width: 100%;
  padding: 10px 12px;
  background-color: #ffffff !important;
  color: #1e293b !important;
  border: 1px solid #cbd5e1;
  border-radius: 6px;
  text-align: center;
  font-size: 15px;
  font-family: inherit;
  box-sizing: border-box;
}
input[type="number"]:focus,
input[type="text"]:focus,
input[type="date"]:focus {
  border-color: #0284c7;
  outline: none;
  box-shadow: 0 0 0 3px rgba(2, 132, 199, 0.15);
}
input[readonly] {
  cursor: not-allowed;
  opacity: 0.8;
}
.table-wrapper {
  overflow-x: hidden;
  margin-bottom: 25px;
}
.responsive-table {
  width: 100%;
  border-collapse: collapse;
  background: white;
}
.responsive-table th {
  background-color: #f1f5f9;
  color: #334155;
  font-weight: 600;
  padding: 14px 10px;
  border-bottom: 2px solid #cbd5e1;
  white-space: nowrap;
}
.responsive-table td {
  padding: 12px 10px;
  border-bottom: 1px solid #e2e8f0;
  vertical-align: middle;
  text-align: center;
}
.item-name {
  text-align: left !important;
  font-weight: 700;
  color: #0f172a;
  white-space: nowrap;
}
.unit-val {
  font-weight: 600;
  color: #0369a1;
  font-size: 16px;
}
.cost-val {
  font-weight: 700;
  color: #1e293b;
  font-size: 16px;
}
.highlight-row td {
  background-color: #f8fafc;
}
.total-row td {
  background-color: #f0f9ff;
  border-bottom: none;
}
.total-label {
  text-align: right !important;
  font-weight: 800;
  color: #0f172a;
  font-size: 16px;
}
.total-amount {
  font-size: 22px;
  font-weight: 800;
  color: #0284c7;
}
.currency {
  font-size: 14px;
  color: #64748b;
  font-weight: normal;
}

.btn-save {
  background: linear-gradient(135deg, #0284c7, #0369a1);
  color: white;
  padding: 16px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  width: 100%;
  font-size: 18px;
  font-weight: 700;
  transition:
    transform 0.2s,
    box-shadow 0.2s;
  box-shadow: 0 4px 12px rgba(2, 132, 199, 0.25);
}
.btn-save:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(2, 132, 199, 0.35);
}
.edit-buttons {
  display: flex;
  gap: 10px;
}
.btn-update {
  flex: 2;
  background: linear-gradient(135deg, #10b981, #059669);
  color: white;
  padding: 16px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 18px;
  font-weight: 700;
}
.btn-cancel-edit {
  flex: 1;
  background: #fee2e2;
  color: #ef4444;
  border: 1px solid #fecaca;
  padding: 16px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 16px;
  font-weight: 700;
}
.btn-update:hover {
  background: linear-gradient(135deg, #059669, #047857);
}
.btn-cancel-edit:hover {
  background: #ef4444;
  color: white;
}

.extra-item-name {
  display: flex;
  align-items: center;
  gap: 10px;
}
.btn-delete-sm {
  background: #fee2e2;
  color: #ef4444;
  border: none;
  width: 30px;
  height: 30px;
  border-radius: 5px;
  cursor: pointer;
  font-weight: bold;
  flex-shrink: 0;
}
.btn-delete-sm:hover {
  background: #ef4444;
  color: white;
}
.btn-add-item {
  background: #f1f5f9;
  color: #0369a1;
  border: 2px dashed #bae6fd;
  padding: 10px 20px;
  border-radius: 8px;
  font-weight: 600;
  cursor: pointer;
  width: 100%;
  transition: 0.2s;
}
.btn-add-item:hover {
  background: #e0f2fe;
  border-color: #38bdf8;
}
.add-item-row td {
  background: transparent;
  padding: 10px 0;
  border: none;
}

.remark {
  margin-top: 25px;
  background-color: #fffbeb;
  border-left: 4px solid #fbbf24;
  padding: 16px;
  border-radius: 0 8px 8px 0;
  font-size: 14px;
  color: #92400e;
}
.remark ul {
  margin: 8px 0 0;
  padding-left: 20px;
  line-height: 1.6;
}
.history-section {
  margin-top: 40px;
  padding-top: 30px;
  border-top: 2px dashed #e2e8f0;
}
.history-section h3 {
  text-align: center;
  color: #0f172a;
  margin-bottom: 20px;
}
.history-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
  gap: 15px;
}
.history-card {
  background: #f8fafc;
  border: 1px solid #e2e8f0;
  border-radius: 10px;
  padding: 16px;
  transition: transform 0.2s;
}
.history-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
}
.history-header {
  display: flex;
  justify-content: space-between;
  border-bottom: 1px solid #e2e8f0;
  padding-bottom: 10px;
  margin-bottom: 10px;
}
.room-highlight {
  font-weight: 700;
  color: #0369a1;
  font-size: 16px;
}
.history-date {
  color: #64748b;
  font-size: 14px;
}
.history-body {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.action-buttons {
  display: flex;
  gap: 5px;
  flex-wrap: wrap;
  justify-content: flex-end;
}
.btn-view {
  background: #e0f2fe;
  color: #0284c7;
  border: 1px solid #bae6fd;
  padding: 6px 10px;
  border-radius: 6px;
  cursor: pointer;
  font-weight: 600;
  font-size: 12px;
}
.btn-edit {
  background: #d1fae5;
  color: #059669;
  border: 1px solid #a7f3d0;
  padding: 6px 10px;
  border-radius: 6px;
  cursor: pointer;
  font-weight: 600;
  font-size: 12px;
}
.btn-delete {
  background: #fee2e2;
  color: #ef4444;
  border: 1px solid #fecaca;
  padding: 6px 10px;
  border-radius: 6px;
  cursor: pointer;
  font-weight: 600;
  font-size: 12px;
}
.btn-view:hover {
  background: #0284c7;
  color: white;
}
.btn-edit:hover {
  background: #059669;
  color: white;
}
.btn-delete:hover {
  background: #ef4444;
  color: white;
}
.empty-state {
  text-align: center;
  padding: 30px;
  color: #94a3b8;
  background: #f8fafc;
  border-radius: 8px;
  border: 1px dashed #cbd5e1;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(15, 23, 42, 0.6);
  backdrop-filter: blur(4px);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  padding: 20px;
  box-sizing: border-box;
}
.modal-content {
  background: white;
  border-radius: 12px;
  width: 100%;
  max-width: 550px;
  max-height: 90vh;
  overflow-y: auto;
  box-shadow:
    0 20px 25px -5px rgba(0, 0, 0, 0.1),
    0 10px 10px -5px rgba(0, 0, 0, 0.04);
  animation: modalIn 0.3s ease-out;
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
  background: #f8fafc;
  padding: 15px 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid #e2e8f0;
}
.modal-header h3 {
  margin: 0;
  font-size: 18px;
  color: #0f172a;
}
.btn-close {
  background: none;
  border: none;
  font-size: 18px;
  cursor: pointer;
  color: #64748b;
}
.btn-close:hover {
  color: #ef4444;
}
.modal-body {
  padding: 25px;
}
.receipt-header {
  border-bottom: 2px dashed #cbd5e1;
  padding-bottom: 15px;
  margin-bottom: 15px;
}
.receipt-header p {
  margin: 5px 0;
  color: #475569;
  font-size: 15px;
}
.highlight-text {
  font-size: 18px;
  font-weight: bold;
  color: #0369a1;
}
.receipt-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 14px;
  margin-bottom: 15px;
  table-layout: fixed;
}
.receipt-table th:nth-child(1),
.receipt-table td:nth-child(1) {
  width: 36%;
  text-align: left;
}
.receipt-table th:nth-child(2),
.receipt-table td:nth-child(2) {
  width: 16%;
  text-align: center;
}
.receipt-table th:nth-child(3),
.receipt-table td:nth-child(3) {
  width: 24%;
  text-align: center;
}
.receipt-table th:nth-child(4),
.receipt-table td:nth-child(4) {
  width: 24%;
  text-align: right;
}
.receipt-table th {
  border-bottom: 1px solid #cbd5e1;
  padding: 10px 0;
  color: #475569;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.receipt-table td {
  padding: 12px 0;
  border-bottom: 1px solid #f1f5f9;
  color: #1e293b;
  overflow: hidden;
  text-overflow: ellipsis;
}
.meter-detail {
  font-size: 12px;
  color: #94a3b8;
  margin-top: 4px;
}
.receipt-table tfoot td {
  border-bottom: none;
  padding-top: 15px;
}
.total-title {
  text-align: right;
  font-weight: 800;
  font-size: 16px;
  padding-right: 15px !important;
  color: #0f172a;
}
.total-price {
  text-align: right;
  font-weight: 800;
  font-size: 18px;
  color: #0284c7;
}
.receipt-footer {
  text-align: center;
  font-size: 12px;
  color: #94a3b8;
  margin-top: 20px;
}
.modal-footer {
  padding: 15px 20px;
  background: #f8fafc;
  border-top: 1px solid #e2e8f0;
  display: flex;
  gap: 10px;
  justify-content: flex-end;
}
.btn-print {
  background: #0ea5e9;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 6px;
  cursor: pointer;
  font-weight: 600;
}
.btn-print:hover {
  background: #0284c7;
}
.btn-cancel {
  background: #e2e8f0;
  color: #475569;
  border: none;
  padding: 8px 16px;
  border-radius: 6px;
  cursor: pointer;
  font-weight: 600;
}
.btn-cancel:hover {
  background: #cbd5e1;
}
.custom-print-date {
  display: none;
}

/* =========================================================
   UI สำหรับหน้าจอมือถือ (ป้องกันหน้าจอทะลุแบบสมบูรณ์)
   ========================================================= */
@media (max-width: 768px) {
  .bill-container {
    padding: 15px;
    margin: 10px;
    overflow-x: hidden;
    width: auto;
    box-sizing: border-box;
  }
  .info-row {
    grid-template-columns: 1fr;
    gap: 15px;
    padding: 0 0 15px 0;
    box-sizing: border-box;
    width: 100%;
    border: none;
    background: transparent;
  }

  .edit-buttons {
    flex-direction: column;
    gap: 10px;
  }
  .btn-update,
  .btn-cancel-edit {
    width: 100%;
    font-size: 16px;
    padding: 14px;
    box-sizing: border-box;
  }

  .responsive-table thead {
    display: none;
  }
  .responsive-table,
  .responsive-table tbody,
  .responsive-table tr,
  .responsive-table td {
    display: block;
    width: 100%;
    box-sizing: border-box;
  }

  .responsive-table tr {
    margin-bottom: 20px;
    background: #ffffff;
    border: 1px solid #e2e8f0;
    border-radius: 10px;
    padding: 15px 10px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
  }

  .responsive-table td {
    display: flex;
    justify-content: space-between;
    align-items: center;
    text-align: right;
    padding: 10px 0;
    border-bottom: 1px dashed #e2e8f0;
    gap: 10px;
    flex-wrap: wrap;
  }
  .responsive-table td:last-child {
    border-bottom: none;
  }

  .responsive-table td::before {
    content: attr(data-label);
    font-weight: 600;
    color: #475569;
    text-align: left;
    flex: 1 1 35%;
    min-width: 0;
    word-break: break-word;
    font-size: 14px;
  }

  .item-name {
    text-align: center !important;
    font-size: 18px;
    color: #0284c7;
    border-bottom: 2px solid #e2e8f0 !important;
    padding-bottom: 15px !important;
    margin-bottom: 10px;
    justify-content: center !important;
    flex-basis: 100% !important;
  }
  .item-name::before {
    display: none;
  }

  .responsive-table td input {
    flex: 1 1 50%;
    max-width: 120px;
    padding: 8px;
    font-size: 14px;
    min-width: 0;
    box-sizing: border-box;
  }

  .unit-val,
  .cost-val,
  .total-amount {
    flex: 1 1 50%;
    text-align: right;
    word-break: break-word;
    min-width: 0;
    font-size: 15px;
  }

  .extra-item-name {
    flex: 1 1 100%;
    display: flex;
    align-items: center;
    justify-content: flex-start;
    gap: 8px;
    min-width: 0;
    margin-bottom: 5px;
  }
  .extra-item-name input {
    width: 100% !important;
    max-width: none !important;
    flex: 1;
  }
  .btn-delete-sm {
    flex-shrink: 0;
  }

  .add-item-row {
    margin-bottom: 10px;
    padding: 0 !important;
    border: none !important;
    box-shadow: none !important;
    background: transparent !important;
  }
  .btn-add-item {
    padding: 12px;
    font-size: 15px;
    width: 100%;
    box-sizing: border-box;
  }

  .hide-on-mobile {
    display: none !important;
  }
  .total-row {
    background: #f0f9ff !important;
    border-color: #bae6fd !important;
  }

  .history-body {
    flex-direction: column;
    align-items: flex-start;
    gap: 15px;
  }
  .action-buttons {
    width: 100%;
    display: grid;
    grid-template-columns: 1fr 1fr 1fr;
    gap: 8px;
    box-sizing: border-box;
  }
  .btn-view,
  .btn-edit,
  .btn-delete {
    text-align: center;
    padding: 10px 5px;
    font-size: 14px;
    width: 100%;
    box-sizing: border-box;
  }

  .modal-content {
    max-width: 95%;
    max-height: 85vh;
    border-radius: 12px;
    width: auto;
  }

  .total-amount {
    font-size: 18px;
    max-width: 55%;
    text-align: right;
    margin-left: auto;
  }
  .currency {
    font-size: 12px;
  }
}

@media print {
  @page {
    margin: 0;
  }
  body,
  html {
    background: white;
    margin: 0;
    padding: 0;
    height: auto;
  }
  .hide-on-print {
    display: none !important;
  }
  .bill-container {
    margin: 0 !important;
    padding: 0 !important;
    box-shadow: none !important;
    background: transparent !important;
  }

  .modal-overlay {
    position: absolute !important;
    top: 0 !important;
    left: 0 !important;
    width: 100% !important;
    height: auto !important;
    background: none !important;
    padding: 0 !important;
    display: block !important;
  }
  .modal-content {
    box-shadow: none !important;
    border: none !important;
    max-width: 100% !important;
    margin: 0 !important;
  }
  .custom-print-date {
    display: block !important;
    position: absolute;
    top: 10mm;
    left: 10mm;
    font-size: 14px;
    color: #000;
  }
  .receipt-slip {
    border: 1px solid #000;
    padding: 30px;
    width: 150mm;
    margin: 25mm auto 0 auto;
  }

  .receipt-header p {
    font-size: 18px;
    margin: 8px 0;
  }
  .highlight-text {
    font-size: 24px;
  }
  .receipt-table {
    font-size: 16px;
  }
  .receipt-table th {
    font-size: 16px;
    padding: 15px 0;
  }
  .receipt-table td {
    font-size: 16px;
    padding: 15px 0;
  }
  .meter-detail {
    font-size: 14px;
    margin-top: 6px;
  }
  .total-title {
    font-size: 18px;
  }
  .total-price {
    font-size: 24px;
  }
  .receipt-footer {
    font-size: 14px;
    margin-top: 30px;
  }
  .receipt-table th,
  .receipt-table td {
    white-space: normal !important;
    overflow: visible !important;
  }
}
</style>
