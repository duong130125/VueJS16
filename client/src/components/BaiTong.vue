<template>
  <div class="container">
    <h1>Quản lý sản phẩm</h1>
    <button class="add-button" @click="openForm()">Thêm mới sản phẩm</button>

    <!-- Overlay và Form thêm/sửa sản phẩm -->
    <div v-if="isFormVisible" class="overlay">
      <div class="form-container">
        <h2>{{ isEditing ? "Cập nhật sản phẩm" : "Thêm sản phẩm" }}</h2>
        <form @submit.prevent="submitForm">
          <div class="form-group">
            <label for="name">Tên sản phẩm</label>
            <input
              v-model.trim="currentProduct.name"
              type="text"
              id="name"
              required
            />
          </div>
          <div class="form-group">
            <label for="image">Hình ảnh (url)</label>
            <input
              v-model.trim="currentProduct.image"
              type="url"
              id="image"
              required
            />
          </div>
          <div class="form-group">
            <label for="price">Giá</label>
            <input
              v-model.number="currentProduct.price"
              type="number"
              id="price"
              min="0"
              required
            />
          </div>
          <div class="form-group">
            <label for="quantity">Số lượng</label>
            <input
              v-model.number="currentProduct.quantity"
              type="number"
              id="quantity"
              min="0"
              required
            />
          </div>
          <div class="form-buttons">
            <button type="button" class="cancel-button" @click="closeForm">
              Hủy
            </button>
            <button type="submit" class="save-button">Lưu</button>
          </div>
        </form>
      </div>
    </div>

    <!-- Bảng sản phẩm -->
    <table class="product-table">
      <thead>
        <tr>
          <th>#</th>
          <th>Tên sản phẩm</th>
          <th>Hình ảnh</th>
          <th>Giá</th>
          <th>Số lượng (kg)</th>
          <th>Ngày thêm</th>
          <th>Chức năng</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(product, index) in products" :key="product.id">
          <td>{{ index + 1 }}</td>
          <td>{{ product.name }}</td>
          <td>
            <img
              :src="product.image"
              :alt="product.name"
              class="product-image"
            />
          </td>
          <td>{{ product.price }} đ</td>
          <td>{{ product.quantity }}</td>
          <td>{{ product.date }}</td>
          <td>
            <button class="action-button edit" @click="editProduct(product)">
              Sửa
            </button>
            <button
              class="action-button delete"
              @click="deleteProduct(product.id)"
            >
              Xóa
            </button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

const products = ref([]);
const isFormVisible = ref(false);
const isEditing = ref(false);
const currentProduct = ref({
  id: null,
  name: "",
  price: "",
  quantity: "",
  image: "",
  date: "",
});

const fetchProducts = async () => {
  try {
    const response = await fetch("http://localhost:8080/products");
    const data = await response.json();
    products.value = data;
  } catch (error) {
    console.error("Error fetching products:", error);
  }
};

const openForm = () => {
  isFormVisible.value = true;
  isEditing.value = false;
  resetForm();
};

const closeForm = () => {
  isFormVisible.value = false;
  resetForm();
};

const resetForm = () => {
  currentProduct.value = {
    id: null,
    name: "",
    price: "",
    quantity: "",
    image: "",
    date: "",
  };
};

const validateForm = () => {
  if (
    !currentProduct.value.name ||
    !currentProduct.value.price ||
    !currentProduct.value.quantity ||
    !currentProduct.value.image
  ) {
    alert("Vui lòng điền đầy đủ thông tin.");
    return false;
  }

  const isDuplicate = products.value.some(
    (product) =>
      product.name.toLowerCase() === currentProduct.value.name.toLowerCase() &&
      product.id !== currentProduct.value.id
  );
  if (isDuplicate) {
    alert("Tên sản phẩm đã tồn tại.");
    return false;
  }

  return true;
};

const submitForm = async () => {
  if (validateForm()) {
    try {
      const url = isEditing.value
        ? `http://localhost:8080/products/${currentProduct.value.id}`
        : "http://localhost:8080/products";
      const method = isEditing.value ? "PUT" : "POST";

      const productToSubmit = {
        ...currentProduct.value,
        date: isEditing.value
          ? currentProduct.value.date
          : new Date().toLocaleDateString("vi-VN"),
      };

      if (!isEditing.value) {
        productToSubmit.id = Date.now();
      }

      const response = await fetch(url, {
        method: method,
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(productToSubmit),
      });

      if (response.ok) {
        if (isEditing.value) {
          const index = products.value.findIndex(
            (p) => p.id === currentProduct.value.id
          );
          if (index !== -1) {
            products.value[index] = productToSubmit;
          }
        } else {
          products.value.push(productToSubmit);
        }
        closeForm();
      } else {
        console.error("Failed to submit product");
        alert(
          `Không thể ${
            isEditing.value ? "cập nhật" : "thêm"
          } sản phẩm. Vui lòng thử lại.`
        );
      }
    } catch (error) {
      console.error("Error submitting product:", error);
      alert(
        `Đã xảy ra lỗi khi ${
          isEditing.value ? "cập nhật" : "thêm"
        } sản phẩm. Vui lòng thử lại.`
      );
    }
  }
};

const editProduct = (product) => {
  isEditing.value = true;
  currentProduct.value = { ...product };
  isFormVisible.value = true;
};

const deleteProduct = async (id) => {
  if (confirm("Bạn có chắc chắn muốn xóa sản phẩm này không?")) {
    try {
      const response = await fetch(`http://localhost:8080/products/${id}`, {
        method: "DELETE",
      });

      if (response.ok) {
        products.value = products.value.filter((product) => product.id !== id);
        console.log("Product deleted successfully");
      } else {
        console.error("Failed to delete product");
        alert("Không thể xóa sản phẩm. Vui lòng thử lại.");
      }
    } catch (error) {
      console.error("Error deleting product:", error);
      alert("Đã xảy ra lỗi khi xóa sản phẩm. Vui lòng thử lại.");
    }
  }
};

onMounted(() => {
  fetchProducts();
});
</script>

<style scoped>
.container {
  font-family: Arial, sans-serif;
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  position: relative;
}

.add-button {
  background-color: #4285f4;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  margin-bottom: 20px;
}

.overlay {
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

.form-container {
  background-color: white;
  padding: 20px;
  border-radius: 8px;
  width: 400px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.form-container h2 {
  margin-top: 0;
  margin-bottom: 20px;
  text-align: center;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

.form-group input {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-sizing: border-box;
}

.form-buttons {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  margin-top: 20px;
}

.cancel-button,
.save-button {
  padding: 8px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.cancel-button {
  background-color: #f1f3f4;
  color: #5f6368;
}

.save-button {
  background-color: #1a73e8;
  color: white;
}

.product-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
}

.product-table th,
.product-table td {
  border: 1px solid #e0e0e0;
  padding: 12px;
  text-align: center;
}

.product-table th {
  background-color: #f8f9fa;
  font-weight: bold;
}

.product-image {
  width: 50px;
  height: 50px;
  object-fit: cover;
}

.action-button {
  padding: 6px 12px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
  margin-right: 5px;
}

.action-button.edit {
  background-color: #ffa000;
  color: white;
}

.action-button.delete {
  background-color: #ff5252;
  color: white;
}
</style>
