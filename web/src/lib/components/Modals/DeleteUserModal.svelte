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
  import Icon from '../Icon.svelte';
  import { fetchRouteApi } from '$lib/api/fetchRouteApi';
  import { showToast } from '../AppToasts.svelte';
  import { customInvalidateAll } from '../PeriodicInvalidator.svelte';
  import { selectedUsersId } from '../../../routes/dashboard/settings/users/+page.svelte';

  interface Props {
    closeModal: CloseModalFn;
    userIds: number[];
    usernames?: string[];
  }

  let { closeModal, userIds, usernames = [] }: Props = $props();

  let deleting = $state(false);

  const deleteUsers = async () => {
    deleting = true;
    let successCount = 0;
    let failCount = 0;

    for (const userId of userIds) {
      const { ok } = await fetchRouteApi({
        method: 'DELETE',
        path: `/users/${userId}`
      });

      if (ok) {
        successCount++;
      } else {
        failCount++;
      }
    }

    deleting = false;

    if (failCount > 0) {
      showToast({
        type: 'error',
        description: `Failed to delete ${failCount} user(s)`,
        duration: 5000
      });
    }

    if (successCount > 0) {
      closeModal(async () => {
        selectedUsersId.set([]);
        await customInvalidateAll();
        await showToast({
          type: 'success',
          description: `${successCount} user(s) deleted.`,
          duration: 3500
        });
      });
    }
  };

  const displayNames = usernames.length > 0
    ? usernames.join(', ')
    : `${userIds.length} user(s)`;
</script>

<ModalBase {closeModal} title="Delete user(s)">
  <div class="flex flex-col min-w-[400px]">
    <div class="flex items-center gap-3 p-4 rounded-lg bg-red-500/10 border border-red-500/30">
      <Icon name="trash" class="w-6 h-6 text-red-500 shrink-0" />
      <p class="text-color text-sm">
        Are you sure you want to delete <strong>{displayNames}</strong>?
        This action cannot be undone.
      </p>
    </div>

    <div class="flex justify-end gap-3 mt-8 w-[350px] ml-auto">
      <Button variant="text" type="button" class="w-2/5" onclick={() => closeModal()}>
        Cancel
      </Button>
      <Button
        variant="containedRed"
        class="w-2/5"
        onclick={deleteUsers}
        disabled={deleting}
      >
        {deleting ? 'Deleting...' : 'Delete'}
      </Button>
    </div>
  </div>
</ModalBase>
