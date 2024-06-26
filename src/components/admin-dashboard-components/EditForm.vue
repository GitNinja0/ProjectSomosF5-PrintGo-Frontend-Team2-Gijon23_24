<script setup>
import axios from "axios";
import { ref, watchEffect, reactive } from "vue";
import { onMounted } from "vue";

const uri = import.meta.env.VITE_API_ENDPOINT_IMAGES;

const props = defineProps({
  onClose: Function,
  productId: Number
});


const closeForm = () => {
  props.onClose();
}

// Product data.

const product = ref();
const productName = ref('');
const price = ref('');
const categoryId = ref('');
const productDescription = ref('');

// Images.

const deletedImages = ref([]);
const addedImages = ref([]);
const imageDirectory = ref([]);
const otherImagesDirectory = ref([])
const mainImage = ref('')
const mainImageUrl = ref('')
const selectedMainImage = ref([])
const selectedFiles = ref([])

// Inputs.

const showMainImageInput = ref(false)

// Find images.

function findImageForProduct(product) {
    const image = product.images.find(img => img.mainImage === true);
    return image
}

function findOtherImagesForProduct(product) {
    const images = product.images.filter(img => img.mainImage === false);
    return images
}

// Handle FILES upload.
const handleFilesChange = (event) => {
  selectedFiles.value = Array.from(event.target.files);
  console.log(selectedFiles.value)
};

// Handle MAIN IMAGE upload.
const handleMainImageChange = (event) => {
  selectedMainImage.value = event.target.files[0];
  console.log(1)
  const reader = new FileReader();
  console.log(reader)
  reader.onload = (e) => {
  console.log(e.target.result)
  mainImageUrl.value = e.target.result;
  reader.readAsDataURL(selectedMainImage.value)
  }
  showMainImageInput.value = false
  console.log(mainImageUrl.value)
  console.log(selectedMainImage.value)
  reader.onerror = (e) => {
    console.error('FileReader error:', e.target.error);
  };
};

// Get product data.

async function getProductData(id) {

  try {
    const response = await axios.get(
      `http://localhost:8080/api/v1/products/${id}`,
      {
        headers: {
          "Content-Type": "application/json",
        },
        withCredentials: true,
      }
    );
    
    product.value = response.data
    productName.value = response.data.productName;
    price.value = response.data.price;
    categoryId.value = response.data.categories[0].id;
    productDescription.value = response.data.productDescription;
    imageDirectory.value.push(findImageForProduct(product.value));
    mainImage.value = imageDirectory.value.imageName
    otherImagesDirectory.value = findOtherImagesForProduct(product.value)
    
      
	} catch (error) {
		console.error("Error al conseguir los datos del producto", error);
		throw error;
  }
}

// Add deleted images to separate array.

const addDeletedImage = (imageId) => {
  const index = otherImagesDirectory.value.findIndex(image => image.id === imageId);
  if (index !== -1) {
    const deletedImage = otherImagesDirectory.value.splice(index, 1)[0];
    deletedImages.value.push(deletedImage);
    console.log(deletedImages.value)
    console.log(otherImagesDirectory.value)
  }
}

function addDeletedMainImage() {
  const deletedMainImage = imageDirectory.value.splice(0, 1)[0];
  showMainImageInput.value = true
  deletedImages.value.push(deletedMainImage);
  console.log(deletedImages.value)
  console.log(showMainImageInput.value)
}


// API Calls handler.

async function handleUpdate() {
	try {
		await updateProduct(props.productId);
    await uploadImages(props.productId);
    for (image in deletedImages) {
      await deleteImages(image.name)
    }
		console.log("Producto actualizado exitosamente.");
	} catch (error) {
		console.error("Error al actualizar el producto", error);
  
		deleteProduct(productId);
	}
}

// Update product data.

async function updateProduct(id) {
	const data = {
		productName: productName.value,
		price: price.value,
		categoryId: categoryId.value,
		productDescription: productDescription.value,
	};
	try {
		const response = await axios.put(
			`http://localhost:8080/api/v1/products/${id}`,
			data,
			{
				headers: {
					"Content-Type": "application/json",
				},
            withCredentials: true,
			}
		);
	} catch (error) {
		console.error("Error al actualizar datos del producto:", error);
		throw error;
	}
}

// Upload new images.

async function uploadImages(id) {
	let formData = new FormData();
	selectedFiles.value.forEach((file) => {
		formData.append("files", file);
	});
	formData.append("file", selectedMainImage.value);
	try {
		await axios.post(
			`http://localhost:8080/api/v1/images/uploadImages/${id}`,
			formData,
			{
				headers: {
					"Content-Type": "multipart/form-data",
				},
				withCredentials: true,
			}
		);

		console.log("Imágenes subidas exitosamente.");
	} catch (error) {
		console.error("Error al subir las imágenes:", error);
		throw error;
	}
}

async function deleteImageByName(name) {
  try {
		await axios.delete(
			`http://localhost:8080/api/v1/images/${name}`,
			{},
			{
				withCredentials: true,
			}
		);

		console.log("Imágen borrada exitosamente.");
	} catch (error) {
		console.error("Error al borrar la imagen:", error);
		throw error;
	}
}




onMounted(() => {
  getProductData(props.productId);
});


</script>

<template>
<Suspense>
  <div class="modal" @click="closeForm">
    <div class="modal_container" @click.stop>

      <div id="button_container">
        <button @click.stop="closeForm">
          <img src="/icons/icon-cross.svg" alt="cross icon">
        </button>
      </div>

      <form @submit.prevent="submitForm">

        <h1>Editar producto</h1>
        <div class="divMain">

          <div class="image-main-container">
            <label>Imagen Principal</label>
            <div class="image-main">
              <input
							type="file"
							class="form-control-file"
							id="file"
							name="file"
							@change="handleMainImageChange"
              v-if="showMainImageInput"
						  />
              <img v-if="!showMainImageInput" :src="mainImageUrl.value" alt="Uploaded Image"> 
              <article v-for="image in imageDirectory" :key="image.id">
                  <div class="delete-image" :style="{ 'background-image': `url('http://localhost:8080/api/v1/imgs/${image.imageName}')` }"></div>
                  <button @click="() => addDeletedMainImage()">Delete</button>
              </article>
            </div>
          </div>
          <section>

              <div class="first-row">

              <div class="title-container">
                <label>Título</label>
                <input type="text" class="form-control" id="title" v-model="productName" placeholder="Título"/>
              </div>

              <div class="price-container">
                <label>Precio</label>
                <input type="number" class="form-control" id="price" v-model="price" placeholder="Precio"/>
              </div>

            </div>

            <div class="categories-container">
              <label>Categoría</label>
              <select id="categories" v-model="categoryId" placeholder="Seleccione categoría">
                <option value=1>Hogar</option>
                <option value=2>Geek</option>
                <option value=3>Litofanía</option>
              </select>

            </div>

          </section>
        </div>

        <div class="existing-images">
          <article v-for="image in otherImagesDirectory" :key="image.id">
            <div class="delete-image" :style="{ 'background-image': `url('http://localhost:8080/api/v1/imgs/${image.imageName}')` }"></div>
            <button @click="() => addDeletedImage(image.id)">Delete</button>
          </article>
        </div>
        <div class="selected-images">
          <article v-for="image in selectedFiles" :key="image.id">
            <div class="delete-image" :style="{ 'background-image': `url('http://localhost:8080/api/v1/imgs/${image.imageName}')` }"></div>
            <button @click="() => addDeletedImage(image.id)">Delete</button>
          </article>
        </div>

        <div class="images-container">
          <label for="file-upload" class="custom-file-upload">Imágenes</label>
          <input type="file" class="form-control-file" id="file-upload"></input>
        </div>
        

        <div class="description-container">
          <label>Descripción</label>
          <textarea class="form-control" id="description" rows="3" v-model="productDescription"
            placeholder="Descripción..."/>
        </div>

        <div class="btns-actions">
          <button @click="handleUpdate">Guardar</button>
        </div>
      </form>
    </div>
  </div>
  </Suspense>
</template>

<style lang="scss" scoped>
@import url('https://fonts.googleapis.com/css2?family=Poppins:ital,wght@0,100;0,200;0,300;0,400;0,500;0,600;0,700;0,800;0,900;1,100;1,200;1,300;1,400;1,500;1,600;1,700;1,800;1,900&display=swap');

.modal {
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  top: 0;
  left: 0;
  height: 100vh;
  width: 100vw;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 1000;
}

.modal_container {
  height: 60rem;
  width: 60rem;
  background-color: white;
  box-shadow: 0 0.2rem 0.4rem rgba(0, 0, 0, 0.1);
  border-radius: 1rem;
  border-color: #AE81D1;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

#button_container {
  width: 100%;
  display: flex;
  padding: 3rem;
  justify-content: end;
}

form {
  display: flex;
  flex-direction: column;
  justify-content: center;
  width: 100%;
 
  max-width: 600px;
  margin: 0 auto;
  padding: 0 2rem 0 2rem;
  border-radius: 1rem;


}

.divMain {
  display: flex;
  justify-content: space-between;
  gap: 1rem;
}

.image-main-container {
  width: 30%;

  input {
    //border: 1px solid black;
    width: 100%;
    height: 8.7rem;
    border-radius: 0.5rem;
    background-color: #DDD8D8;
    font-family: "Poppins", sans-serif;
  }
}
section {
  width: 100%;
}

.first-row {
  display: flex;
  gap: 2rem;
  width: 100%;
  margin-bottom: 1rem;  
}

.title-container {
  width: 70%;

  input {
    width: 100%;
    height: 3rem;
    border-radius: 0.5rem;
    background-color: #DDD8D8;
    font-family: "Poppins", sans-serif;
    font-size: 1rem;
  }
}

.price-container {
  width: 30%;

  input {
    width: 100%;
    height: 3rem;
    border-radius: 0.5rem;
    font-family: "Poppins", sans-serif;
    font-size: 1rem;
    background-color: #DDD8D8;
  }
}

.categories-container {
  display: flex;
  flex-direction: column;
  width: 100%;
  margin-bottom: 1rem;
}

select {
  width: 100%;
  height: 3rem;
  border-radius: 0.5rem;
  //border: 1px solid black;
  background-color: #DDD8D8;
  font-family: "Poppins", sans-serif;
}

.images-container {
  margin-bottom: 1rem;

  input {
    border: 1px solid black;
    width: 100%;
    height: 5rem;
    //border-radius: 0.5rem;
    background-color: #DDD8D8;
    font-family: "Poppins", sans-serif;
  }
}

.description-container {
  width: 100%;
  height: 15rem;

  textarea {
    width: 100%;
    height: 12rem;
    border-radius: 0.5rem;
    //border: 1px solid black;
    background-color: #DDD8D8;
    font-family: "Poppins", sans-serif;
    font-size: 1rem;
  }
}

h1 {
  font-family: "Poppins", sans-serif;
  font-size: 4rem;
  color:grey;
  margin-bottom: 5rem;
  display: flex;
  justify-content: center;
}

label {
  font-family: "Poppins", sans-serif;
  font-size: 1.5rem;
  margin-bottom: 1rem;
}

.btns-actions {
  display: flex;
  justify-content: center;
  gap: 2rem;
  margin-top: 1rem;

  button {
    background-color: #AE81D1;
    font-family: "Poppins", sans-serif;
    font-size: 1.5rem;
    color: white;
    border-radius: 0.5rem;

    width: 8rem;
    height: 4rem;
    display: flex;
    justify-content: center;
    align-items: center;

  }

}

.delete-image {
  height: 8rem;
  width: 8rem;
  background-size: cover;
}

.existing-images {
  display: flex;
  gap: 2rem;
}
</style>