<!--
 Licensed to the Apache Software Foundation (ASF) under one
 or more contributor license agreements.  See the NOTICE file
 distributed with this work for additional information
 regarding copyright ownership.  The ASF licenses this file
 to you under the Apache License, Version 2.0 (the
 "License"); you may not use this file except in compliance
 with the License.  You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing,
 software distributed under the License is distributed on an
 "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
 KIND, either express or implied.  See the License for the
 specific language governing permissions and limitations
 under the License.
-->

<script lang="ts">
  import type { CloseModalFn } from '$lib/types/utilTypes';
  import ModalBase from './ModalBase.svelte';
  import Button from '../Button.svelte';
  import PermissionsManager from '../PermissionsManager.svelte';
  import type { Stream } from '$lib/domain/Stream';
  import { fetchRouteApi } from '$lib/api/fetchRouteApi';
  import { showToast } from '../AppToasts.svelte';
  import { customInvalidateAll } from '../PeriodicInvalidator.svelte';

  interface Props {
    closeModal: CloseModalFn;
    userIds: number[];
    streams: Stream[];
  }

  let { closeModal, userIds, streams }: Props = $props();

  let permissions: any = $state(null);
  let saving = $state(false);

  const savePermissions = async () => {
    if (!permissions) return;
    saving = true;

    let successCount = 0;
    let failCount = 0;

    for (const userId of userIds) {
      const { ok } = await fetchRouteApi({
        method: 'PUT',
        path: `/users/${userId}/permissions`,
        body: {
          permissions
        }
      });

      if (ok) {
        successCount++;
      } else {
        failCount++;
      }
    }

    saving = false;

    if (failCount > 0) {
      showToast({
        type: 'error',
        description: `Failed to update permissions for ${failCount} user(s)`,
        duration: 5000
      });
    }

    if (successCount > 0) {
      closeModal(async () => {
        await customInvalidateAll();
        await showToast({
          type: 'success',
          description: `Permissions updated for ${successCount} user(s).`,
          duration: 3500
        });
      });
    }
  };
</script>

<ModalBase {closeModal} title="Change permissions for {userIds.length} user(s)">
  <div class="flex flex-col min-w-[800px]">
    <PermissionsManager {streams} bind:value={permissions} />

    <div class="flex justify-end gap-3 mt-16 w-[350px] ml-auto">
      <Button variant="text" type="button" class="w-2/5" onclick={() => closeModal()}>
        Cancel
      </Button>
      <Button
        variant="contained"
        class="w-2/5"
        onclick={savePermissions}
        disabled={saving}
      >
        {saving ? 'Saving...' : 'Apply'}
      </Button>
    </div>
  </div>
</ModalBase>
