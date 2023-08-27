<template>
  <div class="wrapper">
    <v-table :columns="columns" :data-source="dataSource">
      <template #favorites="{ record }">
        {{ record.join(',') }}
      </template>
      <template #address="{ record }">
        <span class="red">{{ record }}</span>
      </template>
    </v-table>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'

export default Vue.extend({
  name: 'IndexPage',
  data() {
    return {
      columns: [
        {
          title: 'Name',
          dataIndex: 'name',
        },
        {
          title: 'Age',
          dataIndex: 'age',
        },
        {
          title: 'Infomation',
          dataIndex: 'infomation',

          children: [
            {
              title: 'Contract',
              dataIndex: 'contract',

              children: [
                {
                  title: 'Phone',
                  dataIndex: 'phone',
                },
                {
                  title: 'Email',
                  dataIndex: 'email',
                },
                {
                  title: 'Address',
                  dataIndex: 'address',

                  children: [
                    {
                      title: 'Address',
                      dataIndex: 'address',
                    },
                    {
                      title: 'City',
                      dataIndex: 'city',
                    },
                  ],
                },
              ],
            },
            {
              title: 'Favorites',
              dataIndex: 'favorites',
            },
          ],
        },
      ],
      dataSource: [
        {
          name: 'Van A',
          age: 11,
          infomation: {
            favorites: ['A', 'B', 'C'],
            contract: {
              email: 'example@example.com',
              phone: '(+84) 982 934 000',
              address: {
                city: 'Hanoi',
                address: 'Thach That',
              },
            },
          },
        },
        {
          name: 'Van B',
          age: 12,
          infomation: {
            favorites: ['A', 'B', 'C'],
            contract: {
              email: 'example@example.com',
              phone: '(+84) 982 934 000',
              address: {
                city: 'Hanoi',
                address: 'Dong Truc',
              },
            },
          },
        },
        {
          name: 'Van C',
          age: 13,
          infomation: {
            favorites: ['A', 'B', 'C'],
            contract: {
              email: 'example@example.com',
              phone: '(+84) 982 934 000',
              address: {
                city: 'Hanoi',
                address: 'Ba Dinh',
              },
            },
          },
        },
      ],
    }
  },
})
</script>

<style lang="sass">
.red
  color: red
.blue
  color: blue
</style>
