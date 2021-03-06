<template>
  <div class="grid grid-cols-3 md:grid-cols-7 h-12 items-center p-4">
    <img src="/TFN-black.svg" alt="TF-Logo" class="h-full hidden md:block col-end-3" />
    <img src="/TF-small-black.svg" alt="TF-Logo" class="h-12 md:hidden col-span-1" />
    <div
      class="md:col-end-13 col-span-2 text-right text-gray-500 flex items-center justify-end"
    >
      <AvatarImg :id="user.id" />
      <span class="ml-2">{{ user.id }}</span>
      <button @click="showDialog = true">
        <i class="fas fa-cog text-gray-500"></i>
      </button>
    </div>
    <jdialog v-model="showDialog" @close="showDialog = false" noActions>
      <template v-slot:title>
        <h1>Profile settings</h1>
      </template>
      <div>
        <div
          class="relative flex justify-center h-52"
          @mouseover="showEditPic = true"
          @mouseleave="showEditPic = false"
        >
          <transition name="fade">
            <div
              @click.stop="selectFile"
              v-if="showEditPic"
              class="grid cursor-pointer place-items-center bg-black bg-opacity-75 absolute w-full h-full top-0 left-0"
            >
              <button class="text-white">
                <i class="fas fa-pen"></i>
              </button>
            </div>
          </transition>
          <img class="h-full w-52 bg-black" :src="user.image" />
        </div>
        <h1 class="text-center my-4">{{ user.id }}</h1>
        <div
          class="relative w-full h-full"
          @mouseover="showEdit = true"
          @mouseleave="showEdit = false"
        >
          <transition name="fade">
            <button
              v-if="!isEditingStatus"
              :class="showEdit ? 'block' : 'hidden'"
              class="absolute top-0 right-0"
              @click="setEditStatus(true)"
            >
              <i class="fas fa-pen"></i>
            </button>
          </transition>

          <transition name="fade">
            <button
              v-if="isEditingStatus"
              class="absolute top-1 right-0"
              @click="sendNewStatus"
            >
              <i class="fas fa-check"></i>
            </button>
          </transition>
          <textarea
            v-model="userStatus"
            class="w-full"
            :disabled="!isEditingStatus"
            :placeholder="user.status"
          ></textarea>
        </div>
        <input
          class="hidden"
          type="file"
          id="fileinput"
          ref="fileinput"
          accept="image/*"
          @change="changeFile"
        />
      </div>
    </jdialog>
  </div>
</template>

<script lang="ts">
import { defineComponent, onMounted, ref } from "vue";
import { useAuthState } from "../store/authStore";
import { useSocketActions } from "../store/socketStore";
import Dialog from "./Dialog.vue";
import AvatarImg from "@/components/AvatarImg.vue";

export default defineComponent({
  name: "Topbar",
  components: { AvatarImg, jdialog: Dialog },
  setup() {
    const { user } = useAuthState();
    const showDialog = ref(false);
    const showEdit = ref(false);
    const showEditPic = ref(false);
    const fileinput = ref();
    const file = ref();
    const userStatus = ref("");
    const isEditingStatus = ref(false);

    const selectFile = () => {
      fileinput.value.click();
    };
    const changeFile = () => {
      file.value = fileinput.value?.files[0];
      sendNewAvatar()
    };
    const removeFile = () => {
      file.value = null;
    };

    const sendNewAvatar = async () => {
      const { sendSocketAvatar } = useSocketActions();
      const buffer = await file.value.arrayBuffer();
      sendSocketAvatar(buffer).then(b => alert('refresh for new avatar'));
      showDialog.value = false;
    };

    const setEditStatus = (edit: boolean) => {
      console.log(edit);
      isEditingStatus.value = edit;
      userStatus.value = user.status;
    };
    const sendNewStatus = async () => {
      const { sendSocketUserStatus } = useSocketActions();
      sendSocketUserStatus(userStatus.value);
      user.status = userStatus.value;
      isEditingStatus.value = false;
    };

    return {
      user,
      showEditPic,
      showEdit,
      showDialog,
      fileinput,
      file,
      selectFile,
      changeFile,
      removeFile,
      sendNewAvatar,
      sendNewStatus,
      userStatus,
      setEditStatus,
      isEditingStatus,
    };
  },
});
</script>

<style scoped>
</style>
