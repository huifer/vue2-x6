<template>
<div>
  <span>请求地址: {{url}}</span>
  <span>请求方式: {{method}}</span>


  <div>
    <a-tabs default-active-key="1" >
      <a-tab-pane key="1" tab="请求参数">
        <a-table :columns="columns" :data-source="requestParam">

        </a-table>
      </a-tab-pane>
      <a-tab-pane key="2" tab="响应参数" >
        <a-table :columns="columns" :data-source="responseParam">

        </a-table>
      </a-tab-pane>

    </a-tabs>
  </div>
</div>
</template>

<script>
export default {
  data () {

    return{
      columns: [
        {
          dataIndex: 'field',
          key: 'field',
          title: '参数字段',
          scopedSlots: { customRender: 'field' },
        },
        {
          dataIndex: 'name',
          key: 'name',
          title: '参数名称',
          scopedSlots: { customRender: 'name' },
        },
        {
          dataIndex: 'type',
          key: 'type',
          title: '参数类型',
          scopedSlots: { customRender: 'type' },
        }, {
          dataIndex: 'desc',
          key: 'desc',
          title: '参数说明',
          scopedSlots: { customRender: 'desc' },
        },
      ],
      requestParam: [
        {
          'field': 'username',
          'name': '名称',
          'type': '数字',
          'desc': '用户名',
          "children":[
            {
              'field': 'username',
              'name': '名称',
              'type': '数字',
              'desc': '用户名',
            }
          ]
        }
      ],
      responseParam: [        {
        'field': 'username',
        'name': '名称',
        'type': '数字',
        'desc': '用户名',
        "children":[
          {
            'field': 'username',
            'name': '名称',
            'type': '数字',
            'desc': '用户名',
          }
        ]
      }
      ],
      url: '/baidu.com',
      method:'GET'
    }
  },
  name: 'Execute'
}
</script>

<style scoped>

</style>
