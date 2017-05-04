# vue-datatable

> A datatable component build with Vuejs.
> This is a forked from original [repo](https://github.com/galenyuan/vue-datatable).
> This repo supports Vuejs 2.0

> You can try it [Online](https://galenyuan.github.io/vue-datatable/), it's Vue version [DataTables](https://github.com/DataTables/DataTables)

## Usage
```bash
npm install https://github.com/hrvojevu/vue-datatable.git --save
```
Vue-loader and Babel is required.

```javascript
<data-table :data-table="tableData"></data-table>


import DataTable from 'vue-datatable';

export default {
  computed: {
    ...mapGetters({
      objectList: 'someVuexStore/someGetter' // this can be initialized using Vuex or simply by adding list of objects.
    }),
    rowData: function () {
      return this.objectList.map((object, index) => ({
        id: {
          value: index + 1
        },
        name: {
          value: object.name
        },
        status: {
          value: object.status
        },
        content: {
          value: object.content
        },
        action: {
          value: [
            {
              text: 'Edit',
              class: ['green'],
              func: function () {
              }
            }, {
              text: 'Remove',
              class: ['gray'],
              func: function () {
              }
            }
          ]
        }
      }))
    },
    tableData: function () {
      return {
        options: {
          sortable: true,
          editable: false,
          pageCount: 10
        },
        columns: [
          {
            value: 'id',
            text: '#',
            sortable: true
          },
          {
            value: 'name',
            text: 'Name',
            sortable: true
          },
          {
            value: 'status',
            text: 'Status',
            sortable: true
          },
          {
            value: 'content',
            text: 'Content',
            sortable: false,
            isHTML: true
          },
          {
            value: 'action',
            text: 'Action',
            sortable: false,
            isButton: true
          }
        ],
        rows: this.rowData
      }
    }
  },
  components: {
    DataTable
  }
}
```
