<template>
  <div class="popup">
    <div>
      <header>
        <span>{{ headerTitle }}</span>
      </header>
      <div class="error-message">
        <p>{{ message }}</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "ErrorPopup",
  props: {
    message: String,
    headerTitle: String
  }
};
</script>

<style lang="less" scoped>
.popup {
  position: absolute;
  z-index: 1;
  max-width: 300px;
  width: 100%;

  &:before {
    content: "";
    position: fixed;
    z-index: -1;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    background-color: #38383cbf;
    backdrop-filter: blur(3px);
  }

  .error-message {
    background-color: white;
    border-bottom-left-radius: 32px;
    border-bottom-right-radius: 32px;
    padding: 20px;
    font-weight: 200;
    text-align: center;
  }

  header {
    height: 56px;
    background-color: #f2f4f5;
    border-top-left-radius: 32px;
    border-top-right-radius: 32px;
    padding: 20px;
    font-weight: 400;
    text-align: left;
    color: tomato;
  }
}

</style>
