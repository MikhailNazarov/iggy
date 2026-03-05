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
  import Listbox from '../Listbox.svelte';
  import Input from '../Input.svelte';
  import ModalBase from './ModalBase.svelte';
  import Button from '../Button.svelte';
  import { fetchRouteApi } from '$lib/api/fetchRouteApi';
  import { showToast } from '../AppToasts.svelte';
  import { customInvalidateAll } from '../PeriodicInvalidator.svelte';

  interface Props {
    closeModal: CloseModalFn;
    userId: number;
  }

  let { closeModal, userId }: Props = $props();

  let loading = $state(true);
  let saving = $state(false);
  let username = $state('');
  let selectedStatus = $state<'active' | 'inactive'>('active');
  let usernameError = $state('');

  const loadUser = async () => {
    const { data, ok } = await fetchRouteApi({
      method: 'GET',
      path: `/users/${userId}`
    });

    if (!ok) {
      showToast({
        type: 'error',
        description: 'Failed to load user data',
        duration: 5000
      });
      closeModal();
      return;
    }

    username = data.username;
    selectedStatus = data.status;
    loading = false;
  };

  const saveUser = async () => {
    usernameError = '';
    if (!username.trim()) {
      usernameError = 'Username is required';
      return;
    }

    saving = true;

    const { data, ok } = await fetchRouteApi({
      method: 'PUT',
      path: `/users/${userId}`,
      body: {
        username: username.trim(),
        status: selectedStatus
      }
    });

    saving = false;

    if (!ok) {
      if (data?.field === 'username' && data?.reason) {
        usernameError = data.reason;
      } else {
        const errorMessage = data?.reason || 'Operation failed';
        showToast({
          type: 'error',
          description: errorMessage,
          duration: 5000
        });
      }
      return;
    }

    closeModal(async () => {
      await customInvalidateAll();
      await showToast({
        type: 'success',
        description: `User ${username} has been updated.`,
        duration: 3500
      });
    });
  };

  loadUser();
</script>

<ModalBase {closeModal} title="Edit user">
  {#if loading}
    <div class="h-[100px] flex items-center justify-center">
      <span class="text-color-gray">Loading...</span>
    </div>
  {:else}
    <div class="flex flex-col">
      <div class="grid grid-cols-2 gap-4 min-w-[500px]">
        <Input
          label="Username"
          name="username"
          bind:value={username}
          errorMessage={usernameError}
        />

        <Listbox
          label="Status"
          options={[
            { name: 'Active', value: 'active' },
            { name: 'Inactive', value: 'inactive' }
          ]}
          bind:selectedValue={selectedStatus}
        />
      </div>

      <div class="flex justify-end gap-3 mt-10 w-[350px] ml-auto">
        <Button variant="text" type="button" class="w-2/5" onclick={() => closeModal()}>
          Cancel
        </Button>
        <Button
          variant="contained"
          class="w-2/5"
          onclick={saveUser}
          disabled={saving}
        >
          {saving ? 'Saving...' : 'Save'}
        </Button>
      </div>
    </div>
  {/if}
</ModalBase>
