<template>
  <div class="last-transfers">
    <div class="table-responsive">
      <b-table striped hover :fields="fields" :items="transfers">
        <template #cell(block_number)="data">
          <p class="mb-0">
            <nuxt-link
              v-b-tooltip.hover
              :to="`/block?blockNumber=${data.item.block_number}`"
              title="Check block information"
            >
              #{{ formatNumber(data.item.block_number) }}
            </nuxt-link>
          </p>
        </template>
        <template #cell(from)="data">
          <p class="mb-0">
            <nuxt-link
              :to="`/account/${data.item.from}`"
              :title="$t('pages.accounts.account_details')"
            >
              <Identicon
                :key="data.item.from"
                :address="data.item.from"
                :size="20"
              />
              {{ shortAddress(data.item.from) }}
            </nuxt-link>
          </p>
        </template>
        <template #cell(to)="data">
          <p class="mb-0">
            <nuxt-link
              :to="`/account/${data.item.to}`"
              :title="$t('pages.accounts.account_details')"
            >
              <Identicon
                :key="data.item.to"
                :address="data.item.to"
                :size="20"
              />
              {{ shortAddress(data.item.to) }}
            </nuxt-link>
          </p>
        </template>
        <template #cell(amount)="data">
          <p class="mb-0">
            {{ formatAmount(data.item.amount) }}
          </p>
        </template>
      </b-table>
    </div>
  </div>
</template>

<script>
import gql from 'graphql-tag'
import commonMixin from '@/mixins/commonMixin.js'
import Identicon from '@/components/Identicon.vue'

export default {
  components: {
    Identicon,
  },
  mixins: [commonMixin],
  data() {
    return {
      transfers: [],
      fields: [
        {
          key: 'block_number',
          label: 'Block',
          class: 'd-none d-sm-none d-md-none d-lg-block d-xl-block',
          sortable: true,
        },
        {
          key: 'from',
          label: 'From',
          sortable: true,
        },
        {
          key: 'to',
          label: 'To',
          sortable: true,
        },
        {
          key: 'amount',
          label: 'Amount',
          sortable: true,
        },
      ],
    }
  },
  apollo: {
    $subscribe: {
      extrinsic: {
        query: gql`
          subscription extrinsic {
            extrinsic(
              order_by: { block_number: desc }
              where: {
                section: { _eq: "balances" }
                method: { _like: "transfer%" }
              }
              limit: 10
            ) {
              block_number
              signer
              hash
              args
            }
          }
        `,
        result({ data }) {
          this.transfers = data.extrinsic.map((transfer) => {
            return {
              block_number: transfer.block_number,
              hash: transfer.hash,
              from: transfer.signer,
              to: JSON.parse(transfer.args)[0],
              amount: JSON.parse(transfer.args)[1],
            }
          })
        },
      },
    },
  },
}
</script>

<style>
.last-transfers .table th,
.last-transfers .table td {
  padding: 0.45rem;
}
.last-transfers .table thead th {
  border-bottom: 0;
}
.last-transfers .identicon {
  display: inline-block;
  margin: 0 0.2rem 0 0;
  cursor: copy;
}
</style>
