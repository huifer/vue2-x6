<template>
  <div>

    <div>

      <div>
        <p>当</p>



        <a-row>
          <a-col :span="8">
            <a-select  style="width: 100%" >
            </a-select>
          </a-col>
          <a-col :span="8">
            <a-select  style="width: 100%" >
            </a-select>
          </a-col>
          <a-col :span="8">
            <a-input placeholder="Basic usage" />
          </a-col>
        </a-row>


      </div>

      <a-button type="primary" shape="circle" icon="plus" />

<!--      <a-divider>-->
<!--        <a-select default-value="and" style="width: 120px">-->
<!--          <a-select-option value="and">-->
<!--            且-->
<!--          </a-select-option>-->
<!--          <a-select-option value="or">-->
<!--            或-->
<!--          </a-select-option>-->
<!--        </a-select>-->
<!--      </a-divider>-->
      <a-divider>
        添加一行
      </a-divider>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {}
  },

  name: 'Condition'
}
</script>

<style scoped>

</style>
