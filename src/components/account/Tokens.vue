<template>
  <div class="columns">

    <div class="column col-12">
      <h5>Personal Access Tokens</h5>
    </div>
    <div class="column col-12">
      <div class="divider" />
    </div>
    <div class="column col-12">
      <div class="p-2" />
    </div>
    <div class="column col-12 text-justify">
      <p>
        To access ChaosHub services, you must use a token identifying you on each call you make. This token is a
        <a href="https://tools.ietf.org/html/rfc6750">OAuth2 bearer token</a>
        and is intended to be used against ChaosHub services only.
      </p>
      <p>
        A token is for private use only and should not be shared or made public. If this happens, you may come here and revoke it.
        If you do not need tokens, just revoke existing ones. In that case, you will not be able to access ChaosHub services,
        beyond your history, on this dashboard.
      </p>
    </div>
    <div class="column col-12">
      <div class="p-2" />
      <div class="p-2" />
      <div class="p-2" />
    </div>
    <div class="column col-12">
      <div class="columns">
        <div class="column col-11">
          <h6>Existing Tokens</h6>
          <div class="divider" />
        </div>
        <div class="column col-9">
          <table class="table table-hover">
            <tbody>
              <tr v-for="token in tokens" :key="token.id">
                <td>
                  <div class="columns">
                    <div class="column col-12">
                      <strong>{{token.name}}</strong>
                    </div>
                    <div class="column col-12">
                    <span class="text-gray">{{token.id}}</span>
                    </div>
                    <div class="column col-12" v-if="token.revoked">
                    <span class="text-gray">Revoked</span>
                    </div>
                    <div class="column col-12" v-else>
                    <span class="text-gray">Active</span>
                    </div>
                    <div class="column col-12">
                    <span class="text-gray">Last used: {{renderDate(token.last_used_on)}}</span>
                    </div>
                    <div class="column col-12">
                      <span class="text-gray">Created on: {{renderDate(token.issued_on)}}</span>
                    </div>
                  </div>
                </td>
                <td v-if="!token.revoked">
                  <button class="btn btn-sm float-right" @click.prevent="revokeToken(token.id)">Revoke</button>
                </td>
                <td v-else></td>
                <td>
                  <button class="btn btn-sm btn-error float-right" @click.prevent="deleteToken(token.id)">Delete</button>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
    <div class="column col-12">
      <div class="p-2" />
      <div class="p-2" />
      <div class="p-2" />
    </div>

    <div class="column col-12">
      <h6>New Token</h6>
      <div class="divider col-11" />
      <form>
        <div class="columns">
          <div class="column col-9">
            <div class="form-group">
              <label class="form-label" for="token_name">Name</label>
              <input class="form-input" type="text" id="token_name" placeholder="Name" v-model.trim="new_token.name" />
            </div>
            <div class="form-group">
              <div class="p-2"></div>
              <button class="btn btn-primary" @click.prevent="newToken">Generate Token</button>
            </div>
          </div>
        </div>
      </form>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import axios from 'axios'
import swal from 'sweetalert2'
import moment from 'moment'

export default Vue.extend({
  data() {
      return {
          tokens: [],
          new_token: {
              name: null,
              scopes: null
          }
      }
  },
    created() {
      this.$nextTick(function() {
          this.getAccessTokens()
      })
    },
    methods: {
        renderDate(d: any) {
            if (d == null) {
                return '-'
            }
            if (typeof(d) === 'string') {
                return moment(d).calendar()
            }
            return moment(d * 1000, 'x').calendar()
        },
        newToken() {
          const self = this
          if ( (this.new_token.name === '') || (this.new_token.name === null)) {
              swal({
                    title: `Oops!`,
                    text: `You must provide a name for your token`,
                    buttonsStyling: false,
                    confirmButtonClass: 'btn btn-primary'
                })
              return
          }

          axios.post('/account/user/tokens', self.new_token,
            {
                headers: {
                    'Accept': 'application/json',
                    'Content-Type': 'application/json'
                }
            }
            ).then((response) => {
                const token: any = response.data
                self.tokens.push(token)
                swal({
                    title: `Token Created`,
                    width: 850,
                    html: `Please, keep it safe as this cannot be fetched once
                            this dialog box has been closed:
                            <pre class="code"><code>${token.access_token}</code></pre>`,
                    buttonsStyling: false,
                    confirmButtonClass: 'btn btn-primary'
                })
            }).catch((error) => {
                swal({
                    title: `Failed to generate a new token`,
                    text: error.message,
                    buttonsStyling: false,
                    confirmButtonClass: 'btn btn-primary'
                })
          })
        },
        getAccessTokens() {
            const self = this
            axios.get(
                '/account/user/tokens',
                { headers: { Accept: 'application/json' } }
            ).then((response) => {
                self.tokens = response.data
            }).catch((error) => {
                swal({
                    title: `Failed to retrieve your access tokens!`,
                    text: error.message,
                    buttonsStyling: false,
                    confirmButtonClass: 'btn btn-primary'
                })
            })
        },
        revokeToken(tokenId: string) {
            const self = this
            axios.post(
                '/account/user/tokens/' + tokenId + '/revoke',
                { headers: { Accept: 'application/json' } }
            ).then((response) => {
                self.tokens.forEach((token: any, index: number) => {
                    if (token.id === tokenId) {
                        token.revoked = true
                        return false
                    }
                    return true
                })
                swal({
                    title: `Your token has been revoked!`,
                    text: `You will not be able to use it anymore to converse
                            with the Chaos Platform services.`,
                    buttonsStyling: false,
                    confirmButtonClass: 'btn btn-primary'
                })
            }).catch((error) => {
                swal({
                    title: `Your token failed to be revoked!`,
                    text: error.message,
                    buttonsStyling: false,
                    confirmButtonClass: 'btn btn-primary'
                })
            })
        },
        deleteToken(tokenId: string) {
            const self = this
            axios.delete(
                '/account/user/tokens/' + tokenId,
                { headers: { Accept: 'application/json' } }
            ).then((response) => {
                self.tokens.forEach((token: any, index: number) => {
                    if (token.id === tokenId) {
                        self.tokens.splice(index, 1)
                        return false
                    }
                    return true
                })
                swal({
                    title: `Your token has been deleted!`,
                    text: `You will not be able to use it anymore to converse
                            with the Chaos Platform services.`,
                    buttonsStyling: false,
                    confirmButtonClass: 'btn btn-primary'
                })
            }).catch((error) => {
                swal({
                    title: `Your token failed to be deleted!`,
                    text: error.message,
                    buttonsStyling: false,
                    confirmButtonClass: 'btn btn-primary'
                })
            })
        }
    }
})
</script>
