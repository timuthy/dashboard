<!--
Copyright (c) 2019 by SAP SE or an SAP affiliate company. All rights reserved. This file is licensed under the Apache Software License, v. 2 except as noted otherwise in the LICENSE file

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

     http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

<template>
  <action-icon-dialog
    :shootItem="shootItem"
    @dialogOpened="onConfigurationDialogOpened"
    ref="actionDialog"
    caption="Configure Add-ons"
    maxWidth="900"
    max-height="60vh"
    >
    <template slot="actionComponent">
      <manage-shoot-addons
        ref="addons"
        :isCreateMode="false"
       ></manage-shoot-addons>
    </template>
  </action-icon-dialog>
</template>

<script>
import { mapGetters } from 'vuex'
import ActionIconDialog from '@/dialogs/ActionIconDialog'
import ManageShootAddons from '@/components/ShootAddons/ManageAddons'
import { updateShootAddons } from '@/utils/api'
import { errorDetailsFromError } from '@/utils/error'
import { shootItem } from '@/mixins/shootItem'
import get from 'lodash/get'
import cloneDeep from 'lodash/cloneDeep'

export default {
  name: 'addon-configuration',
  components: {
    ActionIconDialog,
    ManageShootAddons
  },
  props: {
    shootItem: {
      type: Object
    }
  },
  mixins: [shootItem],
  computed: {
    ...mapGetters([
      'isKymaFeatureEnabled'
    ])
  },
  methods: {
    async onConfigurationDialogOpened () {
      this.reset()
      const confirmed = await this.$refs.actionDialog.waitForDialogClosed()
      if (confirmed) {
        this.updateConfiguration()
      }
    },
    async updateConfiguration () {
      try {
        const addons = this.$refs.addons.getAddons()
        await updateShootAddons({ namespace: this.shootNamespace, name: this.shootName, data: addons })
      } catch (err) {
        const errorMessage = 'Could not update addons'
        const errorDetails = errorDetailsFromError(err)
        const detailedErrorMessage = errorDetails.detailedMessage
        this.$refs.actionDialog.setError({ errorMessage, detailedErrorMessage })
        console.error(this.errorMessage, errorDetails.errorCode, errorDetails.detailedMessage, err)
      }
    },
    reset () {
      const addons = cloneDeep(get(this.shootItem, 'spec.addons', {}))
      if (this.isKymaFeatureEnabled) {
        const kymaEnabled = !!get(this.shootItem, 'metadata.annotations["experimental.addons.shoot.gardener.cloud/kyma"]')
        addons['kyma'] = { enabled: kymaEnabled }
      }
      this.$refs.addons.updateAddons(addons)
    }
  }
}
</script>
