<template>
  <div class="v-table">
    <div class="v-table-header">
      <div class="v-table-header-count">
        Show
        <select v-model="pageCount" @change="onChangePageCount()">
          <option>5</option>
          <option>10</option>
          <option>15</option>
          <option>20</option>
        </select>
        items each page
      </div>

      <div class="v-table-header-search">
        <input type="text" v-model="searchBy" class="search" placeholder="Search">
        <button class="reset-search" @click="resetSearch()">Reset</button>
      </div>
    </div>
    <table>
      <thead>
        <tr>
          <th v-for="column in dataTable.columns"
              @click="sortBy(column)"
              :class="{sort: isSortable(column), 
                       desc: sort.sortBy === column.value && sort.desc,
                       asc: sort.sortBy === column.value && !sort.desc}">{{column.text}}</th>
        </tr>
      </thead>

      <tbody>
        <tr v-for="row in paginatedRows" track-by="$index">
          <td v-for="(item, key) in row" @click="editField(item, key)">
            <span v-if="!item.editing">
              <template v-if="isButton(key)">
                <button type="button"
                        v-for="button in item.value"
                        :class="button.class"
                        @click="button.func($event, key, button)">{{button.text}}</button>
              </template>
              <template v-else>
                <template v-if="isHTML(key)"><span v-html="item.value"></span></template>
                <template v-else>{{item.value}}</template>
              </template>
            </span>
            <template v-if="isEditable(item, key)">
              <input type="text" v-model="item.tmpValue">
              <button type="button" @click.stop="saveEdit(item)">Save</button>
              <button type="button" @click.stop="cancelEdit(item)">Cancel</button>
            </template>
          </td>
        </tr>
      </tbody>
    </table>

    <div class="v-table-footer">
      <div class="v-table-footer-info">
        Showing {{firstRow + 1}} to {{lastRow}} of {{filteredRows.length}} items
      </div>

      <div class="v-table-footer-page" v-if="lastPage !== 1">
        <a class="v-table-footer-page-btn" href="javascript:;"
           @click="togglePage('prev')"
           :class="{disabled: currentPage == 1}">Prev</a>
        <a class="v-table-footer-page-btn" href="javascript:;"
           :class="{current: currentPage == 1}"
           @click="togglePage(1)">1</a>
        <span v-if="currentPage >= 5 && lastPage > 10">...</span>
        <a class="v-table-footer-page-btn" href="javascript:;"
           :class="{current: currentPage === page + 1}"
           @click="togglePage(page + 1)"
           v-for="page in centerPartPage">{{page + 1}}</a>
        <span v-if="lastPage > 10 && lastPage - currentPage > 5">...</span>
        <a class="v-table-footer-page-btn" href="javascript:;"
           :class="{current: currentPage === page + 1}"
           @click="togglePage(page + 1)"
           v-for="page in lastPartPage">{{page + 1}}</a>
        <a class="v-table-footer-page-btn" href="javascript:;"
           @click="togglePage('next')"
           :class="{disabled: currentPage === lastPage}">Next</a>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: ['dataTable'],
  data () {
    return {
      pageCount: this.dataTable.options.pageCount,
      currentPage: 1,
      searchBy: '',
      rows: [],
      sort: {
        sortBy: '',
        desc: true
      }
    }
  },
  computed: {
    paginatedRows () {
      return this.getPageRows(this.filteredRows, this.currentPage, this.pageCount)
    },
    filteredRows () {
      return this.filterRows(this.dataTable.rows, this.dataTable.options, this.currentPage)
    },
    lastPage () {
      return Math.ceil(this.filteredRows.length / this.pageCount)
    },
    centerPartPage () {
      if (this.lastPage > 10 && this.currentPage >= 5) {
        if (this.lastPage - this.currentPage > 5) {
          return this.currentPage === this.lastPage ? [this.currentPage - 3, this.currentPage - 2, this.currentPage - 1] : [this.currentPage - 2, this.currentPage - 1, this.currentPage]
        } else {
          const r = []

          for (let i = this.lastPage - 6; i < this.lastPage; i++) {
            r.push(i)
          }
          return r
        }
      } else if (this.lastPage > 10) {
        const r = []

        for (let i = 1; i < 5; i++) {
          r.push(i)
        }
        return r
      } else {
        const r = []

        for (let i = 1; i < this.lastPage; i++) {
          r.push(i)
        }
        return r
      }
    },
    lastPartPage () {
      if (this.lastPage > 10 && this.lastPage - this.currentPage > 5) {
        return [this.lastPage - 1]
      } else {
        return []
      }
    },
    firstRow () {
      return this.currentPage === 1 ? 0 : this.pageCount * (this.currentPage - 1)
    },
    lastRow () {
      return this.pageCount * this.currentPage > this.filteredRows.length ? this.filteredRows.length : this.pageCount * this.currentPage
    }
  },
  watch: {
    'dataTable.rows' (rows) {
      rows.forEach((row, index) => {
        for (let key in row) {
          const column = this.dataTable.columns.filter((column) => {
            return column.value === key
          })[0]

          row[key] = Object.assign({
            editable: column.editable,
            editing: false,
            tmpValue: ''
          }, row[key])
        }

        this.dataTable.rows[index] = row
      })
    },
    'dataTable.columns' (columns) {
      columns.forEach((column, index) => {
        column = Object.assign({
          editable: false,
          sortable: false
        }, column)

        this.dataTable.columns[index] = column
      })
    },
    'searchBy' (val) {
      if (val) {
        this.currentPage = 1
      }
    }
  },
  methods: {
    resetSearch () {
      this.searchBy = ''
    },
    onChangePageCount () {
      this.currentPage = 1
    },
    filterRows (rows, options, currentPage) {
      rows = this.sort.sortBy ? this.sortRows(rows, this.sort.sortBy) : rows

      if (this.searchBy !== '') {
        rows = rows.filter((row) => {
          let r = false

          for (let key in row) {
            if (row[key].value
              .toString()
              .toLowerCase()
              .indexOf(this.searchBy.toLowerCase()) !== -1) {
              r = true
            }
          }

          return r
        })
      }

      return rows
    },
    getPageRows (rows) {
      if (this.currentPage >= this.lastPage && this.lastPage !== 0) {
        let length = rows.slice(this.firstRow, this.lastRow).length
        if (length === 0) {
          this.currentPage -= 1
        }
      }
      return rows.slice(this.firstRow, this.lastRow)
    },
    togglePage (page) {
      switch (page) {
        case 'prev':
          if (this.currentPage <= 1) return
          this.currentPage--
          break
        case 'next':
          if (this.currentPage >= this.lastPage) return
          this.currentPage++
          break
        default:
          if (this.currentPage === page) return
          this.currentPage = page
      }
      if (this.dataTable.onPageChanged) {
        this.dataTable.onPageChanged(this.currentPage)
      }
    },
    sortBy (column) {
      if (!column.sortable || !this.dataTable.options.sortable) return

      if (column.value === this.sort.sortBy) {
        this.sort.desc = !this.sort.desc
      } else {
        this.sort.sortBy = column.value
        this.sort.desc = true
      }
    },
    editField (field, key) {
      if (!this.isEditable(field, key, true)) return

      field.tmpValue = field.value
      field.editing = true
    },
    saveEdit (field) {
      field.value = field.tmpValue
      field.editing = false
      field.tmpValue = ''
    },
    cancelEdit (field) {
      field.editing = false
      field.tmpValue = ''
    },
    sortRows (rows, sortBy) {
      const arr = rows.slice(0)

      return arr.sort((a, b) => {
        const r = this.sort.desc ? a[sortBy].value < b[sortBy].value : a[sortBy].value > b[sortBy].value

        return r ? 1 : -1
      })
    },
    isSortable (column) {
      return column.sortable && this.dataTable.options.sortable
    },
    isEditable (field, key, beforeEditing) {
      const column = this.dataTable.columns.filter((column) => {
        return column.value === key
      })[0]
      if (beforeEditing) {
        return field.editable && this.dataTable.options.editable && column.editable
      } else {
        return field.editable && this.dataTable.options.editable && field.editing && column.editable
      }
    },
    isHTML (key) {
      return this.dataTable.columns.filter((column) => {
        return column.value === key
      })[0].isHTML
    },

    isButton (key) {
      return this.dataTable.columns.filter((column) => {
        return column.value === key
      })[0].isButton
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
  $sortSize: 5px;
  $tableBorderColor: #111111;

  @mixin singleSortIcon($borderColor) {
    content: "";
    position: absolute;
    border-left: $sortSize solid transparent;
    border-right: $sortSize solid transparent;
    border-bottom: 2 * $sortSize solid $borderColor; 
  }

  .v-table {
    button{
      background-color: red;
      border-radius: 5px;
      color: #000;
      border: 1px solid #979797;
      background-color: #CBCCCD;
      margin-left: 5px;
      padding: 5px;
      cursor: pointer;

      &:hover, &:focus {
        outline: none;
      }
    }
    .search{
      border-radius: 5px;
      padding: 7px;
      border: 1px solid #979797;

      &:hover, &:focus {
        outline: none;
      }
    }
    .reset-search{
      font-size: 1rem;
    }
    table {
      width: 100%;
      border-collapse:collapse;

      thead {
        border-bottom: 1px solid $tableBorderColor;

        th {
          position: relative;
          padding: 10px 18px;
          text-align: left;
          background-color: #CBCCCD;
          font-weight: bold;

          &.sort {
            cursor: pointer;

            &::after {
              @include singleSortIcon(#FAFAFA);
              right: $sortSize;
              top: 50%;
              margin-top: -(2 * $sortSize);
            }
            &::before {
              @include singleSortIcon(#FAFAFA);
              right: $sortSize;
              top: 50%;
              margin-top: 3px;
              transform: rotate(180deg);
            }

            &.desc {
              &::after {
                display: none;
              }
              &::before {
                @include singleSortIcon(#333);
                right: $sortSize;
                top: 50%;
                margin-top: -$sortSize;
              }
            }

            &.asc {
              &::before {
                display: none;
              }
              &::after {
                @include singleSortIcon(#333);
                right: $sortSize;
                top: 50%;
                margin-top: -$sortSize;
              }
            }
          }
        }
      }

      tbody {
        border-bottom: 1px solid $tableBorderColor;

        tr {
          background-color: #fff;
          
          td {
            text-align: left;
            padding: 10px 8px;
          }

          &:nth-child(odd) {
            background-color: #f9f9f9;

            td:nth-child(1) {
              background-color: #F1F1F2;
            }
          }

          &:nth-child(even) {
            td:nth-child(1) {
              background-color: #fafafa;
            }
          }
        }
      }
    }
    
    & &-header, & &-footer {
      display: table;
      height: 40px;
      width: 100%;
      line-height: 40px;

      &::after {
        content: '';
        clear: both;
      }
    }

    & &-header {
      &-count {
        float: left;
      }

      &-search {
        float: right;
      }
    }

    & &-footer {
      margin-top: 10px;

      &-info {
        float: left;
      }

      &-page {
        font-size: 0;
        float: right;
        
        span {
          display: inline-block;
          font-size: 1rem;
          padding: 10px 15px;
        }
        
        &-btn {
          display: inline-block;
          height: 40px;
          box-sizing: border-box;
          padding: 0px 10px;
          text-decoration: none;
          color: #000;
          border-radius: 5px;
          border: 1px solid transparent;
          font-size: 1rem;
          
          &:hover {
            color: #000;
            border: 1px solid #979797;
            border-radius: 5px;
            background-color: #CBCCCD;
          }

          &:nth-last-child(1) {
            margin-right: 0;
          }

          &.disabled {
            cursor: default;
            color: #666;

            &:hover {
              color: #666;
              background-color: transparent;
              border: 1px solid transparent;
            }
          }

          &.current {
            color: #000;  
            border: 1px solid #979797;
            border-radius: 5px;
            background-color: #CBCCCD;
          }
        }
      }
    }
  }
</style>
