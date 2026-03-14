<template>
  <div class="text-[16px]">
    <div id="header" class="w-full bg-[#593d88] flex items-center">
      <div
        id="nav"
        class="flex justify-between max-w-7xl w-full py-4 px-8 mx-auto text-white"
      >
        <div id="logo">Redux Shopping Cart</div>
        <div id="cart" class="flex" @click="isShow = !isShow">
          <img src="./assets/images/bag-icon.svg" alt="cart" />
          <span class="bg-[#242526] rounded-full px-2 ml-0.5">{{
            cartNumber
          }}</span>
        </div>
      </div>
    </div>
    <div id="container" class="w-full mx-auto max-w-7xl">
      <div id="home">
        <div id="cards" class="grid xl:grid-cols-4 md:grid-cols-3 gap-8 m-8">
          <div
            id="card"
            class="w-full shadow-[0_5px_15px_rgba(0,0,0,0.35)]"
            v-for="product in products"
          >
            <img :src="product.img" alt="air-pod" class="mx-auto pt-4 w-1/2" />
            <div class="text-[#593d88] w-full text-center mx-auto">
              {{ renderStars(product.stars) }}
            </div>
            <div class="w-full text-center mx-auto my-2.5">
              {{ product.name }}
            </div>
            <div class="w-full text-center mx-auto">
              {{ renderNumber(product.price) }} {{ product.un }}
            </div>
            <div
              class="w-full text-center mx-auto mt-2.5 p-2.5 bg-[#593d88] text-white cursor-pointer"
              @click="handleAddCart(product.id)"
            >
              {{
                clickItemIds[product.id] === product.id
                  ? "Added"
                  : "Add to cart"
              }}
            </div>
          </div>
        </div>
      </div>
    </div>
    <div
      class="fixed z-50 top-0 right-0 h-screen w-screen shadow-[0_5px_15px_rgba(0,0,0,0.35)"
      v-if="isShow"
    >
      <div class="bg-gray-500 opacity-50 inset-0 absolute"></div>
      <div
        class="h-full w-full flex flex-col justify-between lg:w-1/4 right-0 absolute bg-white"
      >
        <div class="w-full bg-[#593d88] flex items-center">
          <div class="flex justify-between w-full py-5 mx-auto text-white">
            <div class="flex" @click="isShow = !isShow">
              <span
                class="bg-white text-black absolute top-0 left-0 py-2.5 px-3 cursor-pointer font-bold"
                >X</span
              >
            </div>
            <div class="mx-auto font-bold">CART ({{ cartNumber }})</div>
          </div>
        </div>
        <div
          class="absolute top-20 w-full text-3xl font-bold"
          v-if="!carts.length"
        >
          Cart is empty
        </div>
        <div class="absolute top-20 w-full" v-else>
          <div class="p-4 grid grid-cols-12 h-32" v-for="item in carts">
            <div class="col-span-5">
              <img :src="item.img" alt="" class="w-full h-24 object-contain" />
            </div>
            <div class="col-span-5">
              <div>{{ item.name }}</div>
              <div>{{ handleEachItemPrice(item.id) }}</div>
            </div>
            <div class="text-center bg-[#593d88] my-auto">
              <div
                class="font-bold bg-[#ebedf0] w-1/2 mx-auto mt-2 cursor-pointer"
                @click="handleEditQuantity('min', item.id)"
              >
                -
              </div>
              <div class="text-white">{{ item.quantity }}</div>
              <div
                class="bg-[#ebedf0] mb-2 w-1/2 mx-auto cursor-pointer"
                @click="handleEditQuantity('add', item.id)"
              >
                +
              </div>
            </div>
            <div
              class="w-full mx-auto text-center my-auto text-2xl font-bold cursor-pointer"
              @click="handleRemoveItem(item.id)"
            >
              x
            </div>
          </div>
        </div>
        <div
          class="w-full bg-[#593d88] px-2 py-2 flex justify-between items-center"
        >
          <div>
            <div class="max-w-5xl text-white">
              Total: {{ renderNumber(handleCalPrice()) }}đ
            </div>
          </div>
          <div>
            <button class="bg-[#ebedf0] py-2 px-2">Checkout</button>
          </div>
        </div>
      </div>
    </div>
    <div id="footer" class="w-full bg-[#593d88] flex items-center">
      <div id="footer-contain" class="max-w-5xl py-2 mx-auto text-white">
        By Sam | from UNICODE.VN
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import type { Cart, Product } from "./types/app.type";

const isShow = ref(false);
const carts = ref<Cart[]>([]);
const cartNumber = ref(0);
const clickItemIds = ref<(number | null)[]>([]);
const handleAddCart = (id: number) => {
  let product = products.find((item) => item.id === id);
  if (!product) return;
  let cartItem = carts.value.find((item) => item.id === id);
  cartItem
    ? cartItem.quantity++
    : carts.value.push({ ...product, quantity: 1 });
  cartNumber.value = handleCountCart(carts.value);
  handleChangeAddCartButton(id);
  return carts;
};
const handleChangeAddCartButton = (id: number) => {
  clickItemIds.value[id] = id;
  setTimeout(() => {
    clickItemIds.value[id] = null;
  }, 3000);
};
const handleCalPrice = () => {
  let total = 0;
  carts.value.forEach((item) => {
    total += item.price * item.quantity;
  });
  return total;
};

const handleEachItemPrice = (id: number) => {
  let cartItem = carts.value.find((item) => item.id === id);
  if (!cartItem) return;
  return renderNumber(cartItem.quantity * cartItem.price) + "đ";
};
const handleCountCart = (obj: { length: number }) => {
  return obj.length;
};
const handleRemoveItem = (id: number) => {
  carts.value = carts.value.filter((item) => item.id != id);
  cartNumber.value = handleCountCart(carts.value);
};

//a: truyền vào chuỗi cộng hoặc trừ
//b: truyền vào ID
const handleEditQuantity = (a: string, b: number) => {
  const cartItem = carts.value.find((item) => item.id === b);
  if (!cartItem) {
    return;
  }
  if (a == "min") {
    if (cartItem.quantity === 1) {
      return (carts.value = carts.value.filter((item) => item.id != b));
    }
    cartItem.quantity--;
  }
  if (a == "add") {
    cartItem.quantity++;
  }
};
const renderNumber = (num: number) => {
  return num.toLocaleString();
};
const renderStars = (star: number) => {
  let s = "";
  for (let index = 0; index < star; index++) {
    s += "★";
  }
  return s;
};
const unit = "đ";
const products: Product[] = [
  {
    id: 1,
    name: "boAt Airdopes 131",
    price: 1099,
    img: "./src/assets/images/air-pod.png",
    stars: 5,
    un: unit,
  },
  {
    id: 2,
    name: "boAt BassHeads 228",
    price: 649,
    img: "./src/assets/images/ear-phone.png",
    stars: 5,
    un: unit,
  },
  {
    id: 3,
    name: "JBL Live 660NC",
    price: 9999,
    img: "./src/assets/images/head-phone.png",
    stars: 3,
    un: unit,
  },
  {
    id: 4,
    name: "boAt Rockerz 255",
    price: 899,
    img: "./src/assets/images/ear-phone-2.png",
    stars: 5,
    un: unit,
  },
  {
    id: 5,
    name: "JBL Endurance Run Sports",
    price: 999,
    img: "./src/assets/images/ear-phone-3.png",
    stars: 5,
    un: unit,
  },
  {
    id: 6,
    name: "JBL Tune 230NC TWS",
    price: 5999,
    img: "./src/assets/images/air-pod-2.png",
    stars: 5,
    un: unit,
  },
  {
    id: 7,
    name: "boAt Rockerz 410",
    price: 1599,
    img: "./src/assets/images/head-phone-2.png",
    stars: 5,
    un: unit,
  },
  {
    id: 8,
    name: "JBL Live 200BT",
    price: 3699,
    img: "./src/assets/images/ear-phone-4.png",
    stars: 5,
    un: unit,
  },
];
</script>
