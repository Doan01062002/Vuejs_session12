<template>
  <div class="container">
    <h1>Quản lý mượn trả sách</h1>

    <button @click="addInfo" class="btn btn-primary mb-3">
      Thêm thông tin
    </button>

    <label for="filter">Lọc theo trạng thái:</label>
    <select v-model="filterStatus" @change="filterBooks" id="filter">
      <option value="">Tất cả</option>
      <option value="Chưa trả">Chưa trả</option>
      <option value="Đã trả">Đã trả</option>
    </select>

    <table class="table table-bordered">
      <thead>
        <tr>
          <th>STT</th>
          <th>Tên sách</th>
          <th>Sinh viên mượn</th>
          <th>Ngày mượn</th>
          <th>Ngày trả</th>
          <th>Trạng thái</th>
          <th>Chức năng</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(item, index) in displayedBooks" :key="index">
          <td>{{ index + 1 }}</td>
          <td>{{ item.title }}</td>
          <td>{{ item.student }}</td>
          <td>{{ formatDate(item.borrowDate) }}</td>
          <td>{{ formatDate(item.returnDate) }}</td>
          <td>
            <select
              v-model="item.status"
              @change="updateStatus(index, item.status)"
            >
              <option value="Chưa trả">Chưa trả</option>
              <option value="Đã trả">Đã trả</option>
            </select>
          </td>
          <td>
            <button
              @click="openEditModal(index)"
              class="btn btn-warning btn-sm"
            >
              Sửa
            </button>
            <button @click="confirmDelete(index)" class="btn btn-danger btn-sm">
              Xóa
            </button>
          </td>
        </tr>
      </tbody>
    </table>

    <!-- Modal Thêm thông tin -->
    <div v-if="isAddModalOpen" class="modal">
      <div class="modal-content">
        <h2>Thêm thông tin mượn sách</h2>
        <label for="newTitle">Tên sách:</label>
        <input type="text" id="newTitle" v-model="newBook.title" required />

        <label for="newStudent">Tên người mượn:</label>
        <input type="text" id="newStudent" v-model="newBook.student" required />

        <label for="newBorrowDate">Ngày mượn:</label>
        <input
          type="date"
          id="newBorrowDate"
          v-model="newBook.borrowDate"
          required
        />

        <label for="newReturnDate">Ngày trả:</label>
        <input
          type="date"
          id="newReturnDate"
          v-model="newBook.returnDate"
          required
        />
        <br />
        <button @click="addBook" class="btn btn-primary">Thêm</button>
        <button @click="closeAddModal" class="btn btn-secondary">Đóng</button>
      </div>
    </div>

    <!-- Modal Cập nhật thông tin -->
    <div v-if="isModalOpen" class="modal">
      <div class="modal-content">
        <h2>Cập nhật tin mượn sách</h2>
        <label for="editTitle">Tên sách:</label>
        <input
          type="text"
          id="editTitle"
          v-model="currentBook.title"
          required
        />

        <label for="editStudent">Tên người mượn:</label>
        <input
          type="text"
          id="editStudent"
          v-model="currentBook.student"
          required
        />

        <label for="editBorrowDate">Ngày mượn:</label>
        <input
          type="date"
          id="editBorrowDate"
          v-model="currentBook.borrowDate"
          required
        />

        <label for="editReturnDate">Ngày trả:</label>
        <input
          type="date"
          id="editReturnDate"
          v-model="currentBook.returnDate"
          required
        />
        <br />
        <button @click="updateBook" class="btn btn-primary">Cập nhật</button>
        <button @click="closeModal" class="btn btn-secondary">Đóng</button>
      </div>
    </div>

    <!-- Modal Xác nhận xóa -->
    <div v-if="isDeleteModalOpen" class="modal">
      <div class="modal-content">
        <h2>Xác nhận xóa</h2>
        <p>
          Bạn có chắc chắn muốn xóa thông tin sách
          {{ bookToDelete.title }} không?
        </p>
        <button @click="deleteBook" class="btn btn-danger">Xóa</button>
        <button @click="closeDeleteModal" class="btn btn-secondary">Hủy</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from "vue";

const books = ref([
  {
    title: "Harry Potter và Hòn Đá Phù Thủy",
    student: "Nguyễn Văn A",
    borrowDate: "2024-04-10",
    returnDate: "2024-04-17",
    status: "Đã trả",
  },
  {
    title: "Lão Hạc",
    student: "Nguyễn Văn B",
    borrowDate: "2024-04-12",
    returnDate: "2024-04-19",
    status: "Chưa trả",
  },
  {
    title: "Chí Phèo",
    student: "Nguyễn Văn C",
    borrowDate: "2024-04-14",
    returnDate: "2024-04-21",
    status: "Đã trả",
  },
]);

const displayedBooks = ref([...books.value]);
const isModalOpen = ref(false);
const isAddModalOpen = ref(false);
const isDeleteModalOpen = ref(false);
const currentBook = ref({});
const newBook = ref({
  title: "",
  student: "",
  borrowDate: "",
  returnDate: "",
  status: "Chưa trả",
});
const bookToDelete = ref(null);
const filterStatus = ref("");

watch(filterStatus, () => {
  filterBooks();
});

// Hàm định dạng ngày tháng năm
const formatDate = (dateString) => {
  const date = new Date(dateString);
  const day = String(date.getDate()).padStart(2, "0");
  const month = String(date.getMonth() + 1).padStart(2, "0");
  const year = date.getFullYear();
  return `${day}/${month}/${year}`;
};

// Thêm mới thông tin mượn trả sách
const addInfo = () => {
  isAddModalOpen.value = true;
};

const addBook = () => {
  const today = new Date().toISOString().split("T")[0];

  // Validate
  if (
    !newBook.value.title ||
    !newBook.value.student ||
    !newBook.value.borrowDate ||
    !newBook.value.returnDate
  ) {
    alert("Tất cả các trường không được phép để trống!");
    return;
  }

  // So sánh ngày mượn và ngày trả
  if (newBook.value.borrowDate < today || newBook.value.returnDate < today) {
    alert("Ngày mượn và ngày trả không được bé hơn ngày hiện tại!");
    return;
  }

  if (newBook.value.borrowDate >= newBook.value.returnDate) {
    alert("Ngày trả phải lớn hơn ngày mượn!");
    return;
  }

  // Thêm vào danh sách và lưu vào localStorage
  books.value.push({ ...newBook.value });
  localStorage.setItem("books", JSON.stringify(books.value));
  closeAddModal();
  filterBooks();
};

const closeAddModal = () => {
  isAddModalOpen.value = false;
};

// Cập nhật trạng thái mượn trả sách
const updateStatus = (index, status) => {
  books.value[index].status = status;
  localStorage.setItem("books", JSON.stringify(books.value));
  filterBooks();
};

// Xóa thông tin mượn trả sách
const confirmDelete = (index) => {
  bookToDelete.value = books.value[index];
  isDeleteModalOpen.value = true;
};

const closeDeleteModal = () => {
  isDeleteModalOpen.value = false;
  bookToDelete.value = null;
};

const deleteBook = () => {
  const index = books.value.indexOf(bookToDelete.value);
  if (index !== -1) {
    books.value.splice(index, 1);
    localStorage.setItem("books", JSON.stringify(books.value));
    closeDeleteModal();
    filterBooks();
  }
};

// Mở modal sửa thông tin
const openEditModal = (index) => {
  currentBook.value = { ...books.value[index] };
  isModalOpen.value = true;
};

// Cập nhật thông tin mượn trả sách
const updateBook = () => {
  const index = books.value.findIndex(
    (book) =>
      book.title === currentBook.value.title &&
      book.student === currentBook.value.student
  );

  // Validate
  if (index !== -1) {
    const today = new Date().toISOString().split("T")[0];

    // Kiểm tra ngày tháng
    if (
      currentBook.value.borrowDate < today ||
      currentBook.value.returnDate < today
    ) {
      alert("Ngày mượn và ngày trả không được bé hơn ngày hiện tại!");
      return;
    }

    if (currentBook.value.borrowDate >= currentBook.value.returnDate) {
      alert("Ngày trả phải lớn hơn ngày mượn!");
      return;
    }

    books.value[index] = { ...currentBook.value };
    localStorage.setItem("books", JSON.stringify(books.value));
    closeModal();
    filterBooks();
  }
};

const closeModal = () => {
  isModalOpen.value = false;
};

// Lọc danh sách theo trạng thái
const filterBooks = () => {
  if (filterStatus.value) {
    displayedBooks.value = books.value.filter(
      (book) => book.status === filterStatus.value
    );
  } else {
    displayedBooks.value = [...books.value];
  }
};

// Khởi tạo dữ liệu từ localStorage
const initBooks = () => {
  const storedBooks = localStorage.getItem("books");
  if (storedBooks) {
    books.value = JSON.parse(storedBooks);
  }
  displayedBooks.value = [...books.value];
};

// Gọi hàm khởi tạo
initBooks();
</script>

<style scoped>
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 5px;
}
</style>
