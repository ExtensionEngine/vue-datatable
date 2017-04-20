# vue-datatable

> A datatable component build with Vuejs.

> You can try it [Online](https://galenyuan.github.io/vue-datatable/), it's Vue version [DataTables](https://github.com/DataTables/DataTables)

##Usage
```bash
npm install --save vue-datatable
```
Vue-loader and Babel is required(maybe will release ES5 and JavaScript file later :P)

```javascript
<data-table :data-table="tableData"></data-table>


import DataTable from 'vue-datatable';

export default {
  data() {
    return {
      tableData: {
        options: {
          sortable: true,
          editable: true,
          pageCount: 10
        },
        columns: [
          {
            value: 'id',
            text: 'ID',
            sortable: true,
            editable: false
          },
          {
            value: 'name',
            text: 'Name',
            sortable: true,
            editable: true
          },
          {
            value: 'age',
            text: 'Age',
            sortable: true,
            editable: true
          },
          {
            value: 'sex',
            text: 'Sex',
            sortable: true,
            editable: true
          },
          {
            value: 'link',
            text: 'Link',
            sortable: false,
            editable: false,
            isHTML: true
          },
          {
            value: 'action',
            text: 'Action',
            sortable: false,
            editable: false,
            isButton: true
          }
        ],
        rows: [],
        onPageChanged (page) {
          console.log('Current page is ' + page)
        }
      }
    }
  },
  created () {
    const chance = new Chance()
    const length = chance.integer({min: 0, max: 30})

    for (let i = 0; i < length; i++) {
      const obj = {
        id: {
          value: i + 1
        },

        name: {
          value: chance.name(),
          editable: chance.bool()
        },
        age: {
          value: chance.age(),
          editable: chance.bool()
        },
        sex: {
          value: chance.gender(),
          editable: chance.bool
        },
        link: {
          value: `<a href="${chance.url()}">${chance.url()}</a>`
        },
        action: {
          value: [
            {
              text: 'action1',
              class: ['red'],
              func: function (event, column, field) {
                console.log('event', event)
                console.log('column', column)
                console.log('field', field)
              }
            },
            {
              text: 'action2',
              class: ['green'],
              func: function (event, column, field) {
                console.log('event', event)
                console.log('column', column)
                console.log('field', field)
              }
            }
          ]
        }
      }
      this.tableData.rows.push(obj)
    }
  },
  components: {
    DataTable
  }
}
```
