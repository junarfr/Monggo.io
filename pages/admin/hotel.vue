<i18n>
{
  "en-us": {
    "wifiName": "@:(wifi) @:(name)",
    "wifiPass": "@:(wifi) @:(password)"
  },
  "en-uk": {
    "wifiName": "@:(wifi) @:(name)",
    "wifiPass": "@:(wifi) @:(password)"
  },
  "id": {
    "wifiName": "@:(name) @:(wifi)",
    "wifiPass": "@:(password) @:(wifi)"
  }
}
</i18n>

<template>
  <app-wrapper
    :title="title"
    :is-dialog="isDialog"
    :is-editing="isEditing"
    :is-confirming="isConfirming"
    :is-deleting="isDeleting"
    :is-previewing="isPreviewing"
    @trigger:refresh="initData"
    @trigger:add="onTriggerAdd"
    @dialog:close="onDialogClose"
    @dialog:action="onDialogAction"
    @delete:close="onDeleteClose"
    @delete:action="onDeleteAction"
    @confirm:close="onConfirmClose"
    @confirm:action="onConfirmAction"
    @preview:close="onPreviewClose"
    @preview:action="onPreviewClose"
  >
    <v-data-table
      :headers="headers"
      :items="items"
      :sort-by="['createdAt']"
      :sort-desc="[true]"
      :loading="isLoading"
    >
      <template #item.image="{ item }">
        <v-avatar :color="getMaterialColor(item.name)" class="ma-1">
          <app-img
            v-if="item.imagesMeta && item.imagesMeta.length > 0"
            :src="item.imagesMeta[0].url"
            :alt="item.imagesMeta[0].name"
          />
          <span
            v-else=""
            :class="{
              'white--text': isDarkColor(getMaterialColor(item.name, true))
            }"
          >
            {{ getInitials(item.name) }}
          </span>
        </v-avatar>
      </template>
      <template #item.phone="{ item }">
        <span>
          {{
            item.callingCodes && item.callingCodes[0] && item.phone
              ? `+${item.callingCodes[0]}${item.phone}`
              : ''
          }}
        </span>
      </template>
      <template #item.createdAt="{ item }">
        <time :datetime="item.createdAt">
          {{
            $moment(item.createdAt)
              .locale($i18n.locale)
              .format('llll')
          }}
        </time>
      </template>
      <template #item.updatedAt="{ item }">
        <time :datetime="item.updatedAt">
          {{
            $moment(item.updatedAt)
              .locale($i18n.locale)
              .format('llll')
          }}
        </time>
      </template>
      <template #item.action="{ item }">
        <v-tooltip bottom="">
          <template #activator="{ on }">
            <v-btn
              :data-cy="`trigger-edit-${slugify(item.name)}`"
              :disabled="isLoading"
              :loading="isLoading"
              class="ma-1"
              color="secondary"
              @click="onTriggerEdit(item)"
              v-on="on"
            >
              <v-icon>mdi-pencil</v-icon>
            </v-btn>
          </template>
          <span>{{ $t('edit') }} {{ item.name }}</span>
        </v-tooltip>
        <v-tooltip bottom="">
          <template #activator="{ on }">
            <v-btn
              :data-cy="`trigger-delete-${slugify(item.name)}`"
              :disabled="isLoading"
              :loading="isLoading"
              class="ma-1"
              color="error"
              @click="onTriggerDelete(item)"
              v-on="on"
            >
              <v-icon>mdi-delete</v-icon>
            </v-btn>
          </template>
          <span>{{ $t('delete') }} {{ item.name }}</span>
        </v-tooltip>
      </template>
    </v-data-table>
    <template #form="">
      <v-text-field
        v-model="item.name"
        v-validate="'required'"
        :error-messages="errors.collect('name')"
        :disabled="isLoading"
        data-vv-name="name"
        :data-vv-as="$t('name')"
        name="name"
        clearable=""
        data-vv-value-path="item.name"
        required=""
        :label="$t('name')"
        outlined=""
      />
      <v-row>
        <v-col cols="12" md="6">
          <v-autocomplete
            v-model="item.callingCodes"
            v-validate="'required'"
            :items="callingCodes"
            :error-messages="errors.collect('callingCode')"
            :disabled="isLoading"
            item-text="name"
            item-value="callingCodes"
            data-vv-name="callingCode"
            :data-vv-as="$t('callingCode')"
            name="callingCode"
            clearable=""
            data-vv-value-path="item.callingCodes"
            required=""
            :label="$t('callingCode')"
            outlined=""
          >
            <template #selection="data">
              <v-chip
                v-bind="data.attrs"
                :input-value="data.selected"
                label=""
                @click="data.select"
              >
                <v-avatar left="" tile="">
                  <app-img
                    v-if="data.item.flag && data.item.flag.length > 0"
                    :src="data.item.flag"
                    :alt="data.item.name"
                  />
                </v-avatar>
                <span>
                  +{{ data.item.callingCodes && data.item.callingCodes[0] }}
                </span>
              </v-chip>
            </template>
            <template #item="{ item }">
              <v-list-item-avatar tile="">
                <app-img
                  v-if="item.flag && item.flag.length > 0"
                  :src="item.flag"
                  :alt="item.name"
                />
              </v-list-item-avatar>
              <v-list-item-content>
                <v-list-item-title>{{ item.name }}</v-list-item-title>
                <v-list-item-subtitle>
                  +{{ item.callingCodes && item.callingCodes[0] }}
                </v-list-item-subtitle>
              </v-list-item-content>
            </template>
          </v-autocomplete>
        </v-col>
        <v-col cols="12" md="6">
          <v-text-field
            v-model="item.phone"
            v-validate="'required|numeric'"
            :error-messages="errors.collect('phone')"
            :disabled="isLoading"
            data-vv-name="phone"
            :data-vv-as="$t('phone')"
            name="phone"
            clearable=""
            data-vv-value-path="item.phone"
            required=""
            :label="$t('phone')"
            outlined=""
          />
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" md="6">
          <v-text-field
            v-model="item.wifiName"
            v-validate="'required'"
            :error-messages="errors.collect('wifiName')"
            :disabled="isLoading"
            data-vv-name="wifiName"
            :data-vv-as="$t('wifiName')"
            name="wifiName"
            clearable=""
            data-vv-value-path="item.wifiName"
            required=""
            :label="$t('wifiName')"
            outlined=""
          />
        </v-col>
        <v-col cols="12" md="6">
          <v-text-field
            v-model="item.wifiPass"
            v-validate="'required'"
            :error-messages="errors.collect('wifiPass')"
            :disabled="isLoading"
            data-vv-name="wifiPass"
            :data-vv-as="$t('wifiPass')"
            name="wifiPass"
            clearable=""
            data-vv-value-path="item.wifiPass"
            required=""
            :label="$t('wifiPass')"
            outlined=""
          />
        </v-col>
      </v-row>
      <v-textarea
        v-model="item.description"
        v-validate="'required'"
        :error-messages="errors.collect('description')"
        :disabled="isLoading"
        data-vv-name="description"
        :data-vv-as="$t('description')"
        name="description"
        clearable=""
        data-vv-value-path="item.description"
        required=""
        :label="$t('description')"
        outlined=""
        auto-grow=""
      />
      <v-file-input
        v-model="item.images"
        v-validate="'required'"
        :error-messages="errors.collect('images')"
        :disabled="isLoading"
        :prepend-icon="null"
        data-vv-name="images"
        :data-vv-as="$tc('image', 2)"
        name="images"
        counter=""
        clearable=""
        data-vv-value-path="item.images"
        required=""
        :label="$tc('image', 2)"
        outlined=""
        multiple=""
        :show-size="1000"
      >
        <template #selection="{ index, text }">
          <v-chip v-if="index < 2" label="" small="">
            {{ text }}
          </v-chip>

          <span
            v-else-if="index === 2"
            class="overline grey--text text--darken-3 mx-2"
          >
            +{{ item.images.length - 2 }}
            {{ $tc('file', item.images.length - 2) }}
          </span>
        </template>
      </v-file-input>
      <v-row>
        <template v-for="meta in item.imagesMeta">
          <v-col :key="meta.url" cols="4" class="d-flex child-flex">
            <v-card
              ripple=""
              flat=""
              color="grey lighten-3"
              @click="onTriggerPreview(meta)"
            >
              <app-img
                v-if="
                  meta.url &&
                    meta.url.length > 0 &&
                    meta.name &&
                    meta.name.length > 0
                "
                :src="meta.url"
                :alt="meta.name"
                :aspect-ratio="1"
              />
            </v-card>
          </v-col>
        </template>
      </v-row>
    </template>
    <template #preview="">
      <app-img
        v-if="
          image.url &&
            image.url.length > 0 &&
            image.name &&
            image.name.length > 0
        "
        :src="image.url"
        :alt="image.name"
      />
    </template>
  </app-wrapper>
</template>

<script>
import { mapState } from 'vuex'
import uuidv4 from 'uuid/v4'
import slugify from '@sindresorhus/slugify'
import _cloneDeep from 'lodash.clonedeep'
import cleanDeep from 'clean-deep'
import isEqual from 'fast-deep-equal'
import isDarkColor from 'is-dark-color'
import materialColorHash from 'material-color-hash'
import initials from 'initials'
import pluralize from 'pluralize'
import paramCase from 'param-case'

import { db, storage } from '~/utils/firebase'

export default {
  layout: 'admin',
  head() {
    return {
      title: `${this.$t(paramCase(this.title))} - Admin`
    }
  },
  data() {
    return {
      title: 'Hotel', // Hold page name
      isDialog: false, // Hold dialog state for editing and adding
      isEditing: false, // Hold editing state
      isConfirming: false, // Hold edit confirmation state
      isDeleting: false, // Hold deleting dialog state
      isPreviewing: false, // Hold previewing dialog state
      isSaved: false, // Hold whether current data is saved
      // Array hold table column
      headers: [
        {
          text: this.$tc('image'),
          value: 'image',
          align: 'center',
          sortable: false
        },
        { text: this.$t('name'), value: 'name' },
        { text: this.$t('phone'), value: 'phone' },
        {
          text: this.$t('createdAt'),
          value: 'createdAt',
          align: 'center',
          sort: (a, b) => {
            return this.$moment(a) - this.$moment(b)
          }
        },
        {
          text: this.$t('updatedAt'),
          value: 'updatedAt',
          align: 'center',
          sort: (a, b) => {
            return this.$moment(a) - this.$moment(b)
          }
        },
        {
          text: this.$t('action'),
          value: 'action',
          align: 'center',
          sortable: false
        }
      ],
      items: [], // Array hold all hotel data
      // Hold hotel data
      item: {
        uid: uuidv4(),
        name: null,
        callingCodes: [],
        phone: null,
        wifiName: null,
        wifiPass: null,
        images: [],
        imagesMeta: [],
        description: null,
        status: null,
        createdAt: null,
        updatedAt: null
      },
      // Hold original hotel data, useful to calculate whether
      // the editing data has changed or not
      itemOriginal: {
        uid: uuidv4(),
        name: null,
        callingCodes: [],
        phone: null,
        wifiName: null,
        wifiPass: null,
        images: [],
        imagesMeta: [],
        description: null,
        status: null,
        createdAt: null,
        updatedAt: null
      },
      // Hold image data
      image: {
        name: '',
        url: '',
        fullPath: '',
        createdAt: ''
      },
      callingCodes: [], // Array hold calling codes data
      // Hold default images meta data
      imagesMeta: [
        {
          createdAt: this.$moment('2019-09-19T02:52:55.000Z').toDate(),
          fullPath: 'default/de65cf77-9923-4fcd-b066-1d02a507c8f6.png',
          name: 'de65cf77-9923-4fcd-b066-1d02a507c8f6.png',
          url:
            'https://firebasestorage.googleapis.com/v0/b/monggo-io.appspot.com/o/default%2Fde65cf77-9923-4fcd-b066-1d02a507c8f6.png?alt=media&token=5704205a-726a-4d40-a4bf-e782cec5b427'
        }
      ]
    }
  },
  computed: {
    ...mapState(['isLoading']),
    collection() {
      return pluralize(paramCase(this.title))
    },
    slugify() {
      return string => {
        if (!string) {
          return
        }
        string.toString()
        return slugify(string)
      }
    },
    isDarkColor() {
      return color => {
        if (!color) {
          return
        }
        color = color.toString()
        return isDarkColor(color)
      }
    },
    getMaterialColor() {
      return (string, isCode = false) => {
        if (!string) {
          return
        }
        string = string.toString()
        const { backgroundColor, materialColorName } = materialColorHash(string)
        return isCode ? backgroundColor : materialColorName.toLowerCase()
      }
    },
    getInitials() {
      return string => {
        if (!string) {
          return
        }
        string = string.toString()
        return initials(string)
      }
    },
    isEdited() {
      const item = _cloneDeep(this.item)
      delete item.refData
      const itemOriginal = _cloneDeep(this.itemOriginal)
      delete itemOriginal.refData
      return this.isEditing && !isEqual(item, itemOriginal)
    }
  },
  watch: {
    'item.images': async function(images) {
      if (images && images.length > 0) {
        const imagesMeta = await Promise.all(
          images.map(async (image, i) => ({
            ...this.item.imagesMeta[i],
            name: image.name,
            url: await this.getUrlFromFile(image)
          }))
        )
        this.item = {
          ...this.item,
          imagesMeta: _cloneDeep(imagesMeta)
        }
      } else {
        this.item.imagesMeta = []
      }
    },
    'itemOriginal.images': async function(images) {
      if (images && images.length > 0) {
        const imagesMeta = await Promise.all(
          images.map(async (image, i) => ({
            ...this.itemOriginal.imagesMeta[i],
            name: image.name,
            url: await this.getUrlFromFile(image)
          }))
        )
        this.itemOriginal = {
          ...this.itemOriginal,
          imagesMeta: _cloneDeep(imagesMeta)
        }
      } else {
        this.itemOriginal.imagesMeta = []
      }
    }
  },
  mounted() {
    this.initData()
  },
  methods: {
    /**
     * Called to get all calling codes information
     */
    async getCallingCodes() {
      try {
        this.$setLoading(true)
        const codes = await this.$http.$get(
          `https://restcountries.eu/rest/v2/all?fields=name;flag;callingCodes;alpha2Code`
        )
        this.callingCodes = codes.filter(
          ({ callingCodes }) =>
            callingCodes.length > 0 && callingCodes[0].length !== 0
        )
      } catch (error) {
        this.$notify({
          isError: true,
          message: error.message
        })
      } finally {
        this.$setLoading(false)
      }
    },
    /**
     * Called to initialize the data
     */
    async initData() {
      try {
        this.$setLoading(true)
        await Promise.all([this.getItems(), this.getCallingCodes()])
      } catch (error) {
        this.$notify({
          isError: true,
          message: error.message
        })
      } finally {
        this.$setLoading(false)
      }
    },
    /**
     * Called to get file form url
     * @param {string} url
     * @param {string} name
     */
    async getFileFromUrl(url, name) {
      try {
        // Taken from: https://stackoverflow.com/questions/44070437/how-to-get-a-file-or-blob-from-an-url-in-javascript
        this.$setLoading(true)
        if (!url) {
          return
        }
        url = url.toString()
        const response = await fetch(url)
        const blob = await response.blob()
        const file = new File([blob], name, {
          type: blob.type
        })
        return file
      } catch (error) {
        this.$notify({
          isError: true,
          message: error.message
        })
      } finally {
        this.$setLoading(false)
      }
    },
    /**
     * Called to get url from file
     * @param {File} file
     */
    getUrlFromFile(file) {
      return new Promise(resolve => {
        const fileReader = new FileReader()
        fileReader.readAsDataURL(file)
        fileReader.addEventListener('load', () => {
          resolve(fileReader.result)
        })
      })
    },
    /**
     * Called to reset data to original form
     */
    reset() {
      const item = {
        uid: uuidv4(),
        name: null,
        callingCodes: [],
        phone: null,
        wifiName: null,
        wifiPass: null,
        images: [],
        imagesMeta: [],
        description: null,
        status: null,
        createdAt: null,
        updatedAt: null
      }
      this.item = _cloneDeep(item)
      this.itemOriginal = _cloneDeep(item)
    },

    /**
     * Called to get all data
     */
    async getItems(collection = this.collection, location, cb) {
      try {
        this.$setLoading(true)
        let snaps = null
        if (typeof collection === 'string') {
          snaps = await db
            .collection(collection)
            .orderBy('createdAt', 'desc')
            .get()
        } else {
          snaps = await collection.get()
        }
        if (snaps && snaps.docs) {
          const items = await Promise.all(
            snaps.docs.map(async doc => {
              const data = doc.data()
              if (cb) {
                const refData = await cb(data)
                return {
                  ...data,
                  refData,
                  imagesMeta: data.imagesMeta.map(meta => ({
                    ...meta,
                    createdAt: meta && meta.createdAt && meta.createdAt.toDate()
                  })),
                  images: [],
                  createdAt: data && data.createdAt && data.createdAt.toDate(),
                  updatedAt: data && data.updatedAt && data.updatedAt.toDate()
                }
              } else {
                return {
                  ...data,
                  imagesMeta: data.imagesMeta.map(meta => ({
                    ...meta,
                    createdAt: meta && meta.createdAt && meta.createdAt.toDate()
                  })),
                  images: [],
                  createdAt: data && data.createdAt && data.createdAt.toDate(),
                  updatedAt: data && data.updatedAt && data.updatedAt.toDate()
                }
              }
            })
          )
          await (() => {
            if (typeof collection === 'string') {
              if (collection === this.collection) {
                this.items = items
              } else if (this[collection]) {
                this[collection] = items
              } else {
                throw new Error('Collection must be defined in the data.')
              }
            } else if (this[location]) {
              this[location] = items
            } else {
              throw new Error('Collection must be defined in the data.')
            }
          })()
        }
      } catch (error) {
        this.$notify({
          isError: true,
          message: error.message
        })
      } finally {
        this.$setLoading(false)
      }
    },
    /**
     * Called to trigger displaying dialog for adding data
     */
    onTriggerAdd() {
      this.$validator.reset()
      this.isDialog = true
    },
    /**
     * Called to trigger displaying dialog for editing data
     */
    async onTriggerEdit(_item) {
      try {
        this.$setLoading(true)
        this.isDialog = true
        this.isEditing = true

        const item = _cloneDeep(_item)

        const images = await Promise.all(
          item.imagesMeta.map(meta =>
            this.getFileFromUrl(meta.url, meta.name || `${uuidv4()}.jpg`)
          )
        )
        item.images = images

        this.item = _cloneDeep(item)
        this.itemOriginal = _cloneDeep(item)
      } catch (error) {
        this.$notify({
          isError: true,
          message: error.message
        })
      } finally {
        this.$setLoading(false)
      }
    },
    /**
     * Called to trigger displaying dialog for deleting data
     */
    async onTriggerDelete(_item) {
      try {
        this.$setLoading(true)
        this.isDeleting = true

        const item = _cloneDeep(_item)

        const images = await Promise.all(
          item.imagesMeta.map(meta =>
            this.getFileFromUrl(meta.url, meta.name || `${uuidv4()}.jpg`)
          )
        )
        item.images = images

        this.item = _cloneDeep(item)
        this.itemOriginal = _cloneDeep(item)
      } catch (error) {
        this.$notify({
          isError: true,
          message: error.message
        })
      } finally {
        this.$setLoading(false)
      }
    },
    /**
     * Called to trigger displaying dialog for previewing image
     */
    onTriggerPreview(item) {
      this.isPreviewing = true
      this.image = _cloneDeep(item)
    },
    /**
     * Called when the user close dialog for adding or editing data
     */
    onDialogClose() {
      if (this.isEdited) {
        if (!this.isSaved) {
          this.isConfirming = true
          return
        }
      }
      this.$validator.reset()
      this.isDialog = false
      this.isEditing = false
      this.isSaved = false
      this.reset()
    },
    /**
     * Called when the user click the save or edit button
     */
    async onDialogAction() {
      try {
        const isValid = await this.$validator.validateAll()
        if (isValid) {
          this.$setLoading(true)
          const date = this.$moment().toDate()
          const payload = {
            ..._cloneDeep(this.item),
            createdAt: this.isEditing ? this.item.createdAt : date,
            updatedAt: date
          }
          if (
            this.isEdited &&
            this.itemOriginal.imagesMeta &&
            this.itemOriginal.imagesMeta.length > 0
          ) {
            await Promise.all(
              this.itemOriginal.imagesMeta.map(meta => {
                if (meta.fullPath.includes('default')) {
                  return
                }
                return storage.ref(meta.fullPath).delete()
              })
            )
          }
          let imagesMeta = this.imagesMeta
          if (this.item.imagesMeta.length > 0) {
            imagesMeta = await Promise.all(
              payload.images.map(async image => {
                const snap = await storage
                  .ref(this.collection)
                  .child(`${uuidv4()}.jpg`)
                  .put(image)
                const url = await snap.ref.getDownloadURL()
                return {
                  url,
                  name: snap.metadata.name,
                  fullPath: snap.metadata.fullPath,
                  createdAt: this.$moment(snap.metadata.timeCreated).toDate()
                }
              })
            )
          }
          payload.imagesMeta = imagesMeta
          if (payload.phone.charAt(0) === '0') {
            payload.phone = payload.phone.substring(1)
          }
          delete payload.images
          delete payload.refData
          this.isSaved = true
          await db
            .collection(this.collection)
            .doc(payload.uid)
            .set(
              cleanDeep(payload, {
                emptyArrays: false,
                emptyObjects: false,
                emptyStrings: false,
                nullValues: false
              }),
              { merge: true }
            )
          await this.getItems()
          await this.onDialogClose()
          await this.$notify({ kind: 'success', message: this.$t('dataSaved') })
        }
      } catch (error) {
        console.log(error)
        this.$notify({
          isError: true,
          message: error.message
        })
      } finally {
        this.$setLoading(false)
      }
    },
    /**
     * Called when the user close delete confirmation dialog
     */
    onDeleteClose() {
      this.$validator.reset()
      this.isDeleting = false
      this.reset()
    },
    /**
     * Called when the user click the delete button on dialog
     */
    async onDeleteAction() {
      try {
        this.$setLoading(true)
        const item = _cloneDeep(this.item)

        await db
          .collection(this.collection)
          .doc(item.uid)
          .delete()
        if (item.imagesMeta && item.imagesMeta.length > 0) {
          await Promise.all(
            item.imagesMeta.map(meta => {
              if (meta.fullPath.includes('default')) {
                return
              }
              return storage.ref(meta.fullPath).delete()
            })
          )
        }
        const [
          roomsRef,
          qrRef,
          servicesRef,
          ordersRef,
          usersRef
        ] = await Promise.all([
          db
            .collection('rooms')
            .where('hotel', '==', item.uid)
            .get(),
          db
            .collection('qr-codes')
            .where('hotel', '==', item.uid)
            .get(),
          db
            .collection('services')
            .where('hotel', '==', item.uid)
            .get(),
          db
            .collection('orders')
            .where('hotel', '==', item.uid)
            .get(),
          db
            .collection('users')
            .where('hotel', '==', item.uid)
            .get()
        ])
        // Delete Image
        await Promise.all([
          roomsRef.docs.map(async roomDoc => {
            const room = roomDoc.data()
            if (room && room.imagesMeta && room.imagesMeta.length > 0) {
              await Promise.all(
                room.imagesMeta.map(meta => {
                  if (meta.fullPath.includes('default')) {
                    return
                  }
                  return storage.ref(meta.fullPath).delete()
                })
              )
            }
          }),
          servicesRef.docs.map(async serviceDoc => {
            const service = serviceDoc.data()
            if (
              service &&
              service.imagesMeta &&
              service.imagesMeta.length > 0
            ) {
              await Promise.all(
                service.imagesMeta.map(meta => {
                  if (meta.fullPath.includes('default')) {
                    return
                  }
                  return storage.ref(meta.fullPath).delete()
                })
              )
            }
          })
        ])
        // Delete relation
        await Promise.all([
          ...roomsRef.docs.map(roomDoc => roomDoc.ref.delete()),
          ...qrRef.docs.map(qrDoc => qrDoc.ref.delete()),
          ...servicesRef.docs.map(serviceDoc => serviceDoc.ref.delete()),
          ...ordersRef.docs.map(orderDoc => orderDoc.ref.delete()),
          ...usersRef.docs.map(userDoc => userDoc.ref.delete())
        ])
        await this.getItems()
        await this.onDeleteClose()
        await this.$notify({ kind: 'success', message: this.$t('dataDeleted') })
      } catch (error) {
        this.$notify({
          isError: true,
          message: error.message
        })
      } finally {
        this.$setLoading(false)
      }
    },
    /**
     * Called when the user close edit confirmation dialog
     */
    onConfirmClose() {
      this.isConfirming = false
    },
    /**
     * Called when the user click yes in edit confirmation dialog
     */
    onConfirmAction() {
      this.onConfirmClose()
      this.$validator.reset()
      this.isDialog = false
      this.isEditing = false
      this.reset()
    },
    /**
     * Called when the user close image preview
     */
    onPreviewClose() {
      this.isPreviewing = false
      this.image = {
        name: '',
        url: '',
        fullPath: '',
        createdAt: ''
      }
    },
    /**
     * Called when the user click ok on image preview
     */
    onPreviewAction() {
      this.onPreviewClose()
    }
  }
}
</script>
