<template>
  <div>
    <h2>ListProducts</h2>

    <!-- Button để lấy chi tiết sản phẩm dựa trên ID -->
    <button @click="getProductById">Get detail</button>

    <!-- Button để xóa sản phẩm dựa trên ID -->
    <button @click="removeProductById">Delete</button>

    <!-- Button để thêm mới sản phẩm -->
    <button @click="createProduct">Create Product</button>

    <!-- Button để cập nhật sản phẩm dựa trên ID -->
    <button @click="updateProductById">Update Product</button>

    <!-- Hiển thị danh sách sản phẩm -->
    <ul>
      <li v-for="product in products" :key="product.id">
        {{ product.name }} - {{ product.price }}
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

// Khai báo biến products để lưu trữ danh sách sản phẩm
const products = ref([]);

// Hàm để lấy toàn bộ sản phẩm từ API và lưu vào biến products
const getAllProduct = async () => {
  try {
    const response = await fetch("http://localhost:8080/products");
    const data = await response.json();
    products.value = data;
    console.log("Danh sách sản phẩm:", products.value);
  } catch (error) {
    console.error("Không thể tải sản phẩm:", error);
  }
};

// Hàm để lấy thông tin chi tiết của một sản phẩm dựa trên id
const getProductById = async () => {
  const id = 5; // Thay đổi id của sản phẩm cần lấy chi tiết
  try {
    const response = await fetch(`http://localhost:8080/products/${id}`);
    if (!response.ok) {
      console.log("Không tìm thấy bản ghi");
      return;
    }
    const data = await response.json();
    console.log("Chi tiết sản phẩm:", data);
  } catch (error) {
    console.error("Lỗi khi lấy thông tin sản phẩm:", error);
  }
};

// Hàm để xóa sản phẩm dựa trên id
const removeProductById = async () => {
  const id = 5; // Thay đổi id của sản phẩm cần xóa
  try {
    const response = await fetch(`http://localhost:8080/products/${id}`, {
      method: "DELETE",
    });
    if (!response.ok) {
      console.log("Không thể xóa bản ghi");
      return;
    }
    getAllProduct();
  } catch (error) {
    console.error("Lỗi khi xóa sản phẩm:", error);
  }
};

// Hàm để tạo mới sản phẩm
const createProduct = async () => {
  const product = {
    name: "GPS 5",
    image: "gps5.jpg",
    price: 300,
    quantity: 15,
  };

  try {
    const response = await fetch("http://localhost:8080/products", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(product),
    });

    if (!response.ok) {
      console.log("Không thể tạo sản phẩm mới");
      return;
    }

    const result = await response.json();
    console.log("Kết quả tạo sản phẩm từ server:", result);
    getAllProduct();
  } catch (error) {
    console.error("Lỗi khi tạo sản phẩm:", error);
  }
};

// Hàm để cập nhật thông tin sản phẩm theo id
const updateProductById = async () => {
  const id = 5; // Thay đổi id của sản phẩm cần cập nhật
  const product = {
    name: "RobotGunDam",
    image: "robot.jpg",
    price: 350,
    quantity: 20,
  };

  try {
    const response = await fetch(`http://localhost:8080/products/${id}`, {
      method: "PUT",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(product), // Truyền đối tượng product vào body của fetch
    });

    if (!response.ok) {
      console.log("Không thể cập nhật sản phẩm");
      return;
    }

    const result = await response.json();
    console.log("Kết quả cập nhật sản phẩm từ server:", result);

    // Cập nhật lại danh sách sản phẩm sau khi cập nhật
    getAllProduct();
  } catch (error) {
    console.error("Lỗi khi cập nhật sản phẩm:", error);
  }
};

onMounted(() => {
  getAllProduct();
});
</script>

<style scoped></style>
