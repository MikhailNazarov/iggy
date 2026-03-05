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
    userId: number;
    streams: Stream[];
  }

  let { closeModal, userId, streams }: Props = $props();

  let permissions: any = $state(null);
  let initialPermissions: any = $state(null);
  let loading = $state(true);
  let saving = $state(false);
  let username = $state('');

  const loadUser = async () => {
    const { data, ok } = await fetchRouteApi({
      method: 'GET',
      path: `/users/${userId}`
    });

    if (!ok) {
      showToast({
        type: 'error',
        description: 'Failed to load user permissions',
        duration: 5000
      });
      closeModal();
      return;
    }

    username = data.username;
    initialPermissions = data.permissions;
    loading = false;
  };

  const savePermissions = async () => {
    if (!permissions) return;
    saving = true;

    const { data, ok } = await fetchRouteApi({
      method: 'PUT',
      path: `/users/${userId}/permissions`,
      body: {
        permissions
      }
    });

    saving = false;

    if (!ok) {
      const errorMessage = data?.reason || 'Failed to update permissions';
      showToast({
        type: 'error',
        description: errorMessage,
        duration: 5000
      });
      return;
    }

    closeModal(async () => {
      await customInvalidateAll();
      await showToast({
        type: 'success',
        description: `Permissions for ${username} have been updated.`,
        duration: 3500
      });
    });
  };

  loadUser();
</script>

<ModalBase {closeModal} title={username ? `Edit user permissions — ${username}` : 'Edit user permissions'}>
  {#if loading}
    <div class="h-[100px] flex items-center justify-center">
      <span class="text-color-gray">Loading...</span>
    </div>
  {:else}
    <div class="flex flex-col min-w-[800px]">
      <PermissionsManager {streams} bind:value={permissions} {initialPermissions} />

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
          {saving ? 'Saving...' : 'Save'}
        </Button>
      </div>
    </div>
  {/if}
</ModalBase>
