<template>
  <div v-if="prop">
    <h4>{{prop.name}}</h4>
    <hr>
    <li class="productPrice">{{localePrice}} 원</li>
    <li class="deliveryCharge">{{localeCharge!='0'?'배송비 : '+localeCharge+' 원':'무료배송'}}</li>
    <br>
    <b-form-select v-model="selectedSize" value="null" id="size" :options="sizeOp" class="mb-3"/>
    <b-form-select v-model="selectedColor" value="null" id="color" :options="colorOp" class="mb-3"/>
    <b-form-input v-model="selectedCount" type="number" min="1" max="20"></b-form-input>

    <br>
    <div class="row">
      <div class="col-6">
        <button @click="goCart" type="button" class="btn btn-color2 detailBtn">장바구니에 담기</button>
      </div>
      <div class="col-6">
        <button @click="goOrder" type="button" class="btn btn-color1 detailBtn">바로구매</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  computed: {
    // 숫자 세자리 단위에서 ,를 표시한다.
    localePrice() {
      return Number(this.prop.price).toLocaleString("en");
    },
    localeCharge() {
      return Number(this.prop.deliveryCharge).toLocaleString("en");
    }
  },
  data() {
    return {
      selectedCount: 1,
      selectedSize: null,
      selectedColor: null,
      sizeOp: [
        { value: null, text: "사이즈" },
        { value: "S", text: "S" },
        { value: "M", text: "M" },
        { value: "L", text: "L" }
      ],
      colorOp: [
        { value: null, text: "색상" },
        { value: "charcoal", text: "챠콜" },
        { value: "black", text: "블랙" },
        { value: "white", text: "화이트" }
      ]
    };
  },
  methods: {
    // 로그인 여부와 옵션 선택 여부 확인 후,
    // spring CartController @PostMapping("/cart")에 데이터를 전달한다.
    // 그 후   self.$router.push("/cart")로 이동한다.
    goCart() {
      var self = this;

      if (sessionStorage.getItem("user") === null) {
        alert("회원만 구매가능한 상품입니다. 로그인 화면으로 이동합니다.");
        this.$router.push("/login");
        return;
      } else if (
        this.selectedCount !== null &&
        (this.selectedSize !== null && this.selectedColor !== null)
      ) {
        this.$http
          .post("/cart", {
            user_email: JSON.parse(sessionStorage.getItem("user")).email,
            product_id: self.prop.id,
            cart_size: self.selectedSize,
            cart_color: self.selectedColor,
            cart_count: Number(self.selectedCount)
          })
          .then(function(response) {
            self.$router.push("/cart");
          })
          .catch(function(error) {
            console.log(error);
          });
      } else {
        alert("옵션을 모두 선택해주세요.");
      }
    },

    // 로그인 여부와 옵션 선택 여부 확인 후,
    // color, size, count 수정 후 prop을 세션에 저장한다.
    // 그 후  self.$router.push("/order")로 이동한다.
    //
    goOrder() {
      this.prop.color = this.selectedColor;
      this.prop.size = this.selectedSize;
      this.prop.hitcount = this.selectedCount;

      if (sessionStorage.getItem("user") === null) {
        alert("회원만 구매가능한 상품입니다. 로그인 화면으로 이동합니다.");
        this.$router.push("/login");
      } else if (
        this.selectedCount !== null &&
        (this.selectedSize !== null && this.selectedColor !== null)
      ) {
        sessionStorage.setItem("order", JSON.stringify([this.prop]));
        this.$router.push("/order");
      } else {
        alert("옵션을 모두 선택해주세요.");
      }
    }
  },
  props: ["prop"]
};
</script>

<style>
li {
  list-style: none;
}

.productPrice {
  font-size: 1.2em;
}

.deliveryCharge {
  font-size: 0.8em;
}
.detailBtn{
  width: 100% ;
  height: 100% ;
}
</style>
