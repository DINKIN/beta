<template>
  <div
    :class="{
      deleted: post.is_deleted,
      'h-entry': detail
    }"
    class="media w-100 justify-content-start"
  >
    <a
      v-if="detail"
      v-show="false"
      :href="(mainPost.user.verified ? mainPost.user.verified.link : `/@${mainPost.user.username}`)"
      rel="author"
      class="p-author h-card">
      <avatar
        :avatar="mainPost.user.content.avatar_image"
        :alt="(mainPost.user.name || mainPost.user.username)"
        class="d-flex mr-3 iconSize u-photo"
        size="64"
        max-size="64" />
    </a>
    <nuxt-link
      v-if="!preview"
      :to="`/@${mainPost.user.username}`">
      <avatar
        :avatar="mainPost.user.content.avatar_image"
        :alt="mainPost.user.username"
        class="d-flex mr-3 iconSize"
        size="64"
        max-size="64" />
    </nuxt-link>
    <div class="media-body">
      <h6 class="mt-1">
        <nuxt-link
          :to="`/@${mainPost.user.username}`"
          class="text-gray-dark">
          {{ mainPost.user.username }}
          <emojify
            v-if="mainPost.user.name"
            :text="mainPost.user.name"
            element="small"
            class="text-muted"
          />
        </nuxt-link>
      </h6>
      <div class="d-flex flex-wrap flex-md-nowrap">
        <p
          :class="{
            'mb-0': preview,
            'e-content p-name': detail
          }"
          class="flex-grow-1 w-100"
        >
          <entity-text
            :content="mainPost.content"
            :spoiler="spoiler"
            :longpost="longpost"
            :deleted="post.is_deleted">
            <em>[Post deleted]</em>
          </entity-text>
        </p>
        <div
          v-if="thumbs.length"
          class="flex-shrink-1 mb-2 d-flex mr-auto ml-auto mr-md-2 flex-wrap flex-lg-nowrap justify-content-md-end">
          <thumb
            v-for="(t, i) in thumbs"
            :original="t.original"
            :thumb="t.thumb"
            :original-width="t.width"
            :original-height="t.height"
            :key="i"
            class="mx-1 mb-1 mb-lg-0" />
        </div>
        <div
          v-if="clips.length"
          class="flex-shrink-1 mb-2 d-flex mr-auto ml-auto mr-md-2 flex-wrap flex-lg-nowrap justify-content-md-end">
          <sound
            v-for="(t, i) in clips"
            :url="t.url"
            :title="t.title"
            :key="i" />
        </div>
      </div>
      <div
        v-if="poll"
        class="card mb-3">
        <div class="card-body">
          <poll
            :poll="poll"
            :poll-id="poll.poll_id"
            :poll-token="poll.poll_token" />
        </div>
      </div>
      <p v-if="channelInvite">
        <nuxt-link
          :to="`/messages/${channelInvite.channel_id}`"
          class="btn btn-outline-primary"
        >
          Go to the channel {{ channelInvite.channel_id }}
        </nuxt-link>
      </p>
      <footer v-if="!post.is_deleted && !preview">
        <div v-if="post.repost_of">
          <nuxt-link
            :to="`/@${post.user.username}`"
            class="text-muted">
            <font-awesome-icon
              icon="retweet"
              class="mr-1"
            />
            <span>Reposted by @{{ post.user.username }}</span>
          </nuxt-link>
        </div>
        <ul class="list-inline">
          <li class="list-inline-item">
            <nuxt-link
              ref="link"
              :to="permalink"
              :class="{ 'u-url': detail }"
              :title="absDate"
              class="text-muted">
              <font-awesome-icon
                :icon="['far', 'clock']"
                class="mr-1"
              />
              <time
                :class="{ 'dt-published': detail }"
                :datetime="absDate">{{ detail ? absDate : date }}</time>
            </nuxt-link>
          </li>
          <template v-if="mainPost.is_revised">
            <li class="list-inline-item revised">
              <template v-if="mainPost.revision">
                <font-awesome-icon
                  :icon="['far', 'edit']"
                  class="mr-1"
                />
                Original
              </template>
              <template v-else>
                <nuxt-link
                  :to="revisions_permalink"
                  class="text-muted"
                  title="Revised Post">
                  <font-awesome-icon
                    icon="edit"
                    class="mr-1"
                  />
                  <span>Revised</span>
                </nuxt-link>
              </template>
            </li>
          </template>
          <template v-if="post.reply_to">
            <li class="list-inline-item">
              <nuxt-link
                :to="reply_permalink"
                :class="{ 'u-in-reply-to': detail }"
                class="text-muted"
                title="In Reply To">
                <font-awesome-icon
                  icon="comments"
                  class="mr-1"
                />
              </nuxt-link>
            </li>
          </template>
          <template v-else-if="post.counts.replies">
            <li class="list-inline-item">
              <nuxt-link
                :to="permalink"
                class="text-muted"
                title="Thread Starter">
                <font-awesome-icon :icon="['far', 'comments']"/>
              </nuxt-link>
            </li>
          </template>
          <template v-if="!viewOnly && user">
            <li class="list-inline-item reply">
              <a
                class="text-muted"
                href="#"
                @click.prevent="replyModal"
              >
                <font-awesome-icon
                  icon="reply"
                  class="mr-1"
                />
                <span>Reply</span>
              </a>
            </li>
          </template>
          <template v-if="!viewOnly && me">
            <li class="list-inline-item remove">
              <a
                class="text-muted"
                href="#"
                @click.stop.prevent="removeModal">
                <font-awesome-icon
                  icon="trash"
                  class="mr-1"
                />
                <span>Remove</span>
              </a>
            </li>
          </template>
          <template v-if="!viewOnly || !user">
            <li class="list-inline-item source">
              <a
                :href="post.source.link"
                class="text-muted"
                target="_new">
                <font-awesome-icon
                  icon="paper-plane"
                  class="mr-1"
                />
                <span>via {{ mainPost.source.name }}</span>
              </a>
            </li>
          </template>
          <template v-if="crosspost">
            <li class="list-inline-item crosspost-url">
              <a
                :href="crosspost"
                class="text-muted"
                target="_new">
                <font-awesome-icon
                  icon="random"
                  class="mr-1"
                />
                <span>Crosspost</span>
              </a>
            </li>
          </template>
        </ul>
      </footer>
      <template v-if="detail">
        <hr>
        <div class="d-flex align-items-center">
          <ul class="list-inline">
            <li class="list-inline-item">
              <div class="count">
                {{ post.counts.replies }}
              </div>
              <small class="text-muted">replies</small>
            </li>
            <li class="list-inline-item">
              <div class="count">
                {{ post.counts.reposts }}
              </div>
              <small class="text-muted">reposts</small>
            </li>
            <li class="list-inline-item">
              <div class="count">
                {{ post.counts.bookmarks }}
              </div>
              <small class="text-muted">stars</small>
            </li>
          </ul>
          <ul class="list-inline ml-3">
            <li
              v-for="user in reactionUsers"
              :key="user.id"
              class="list-inline-item">
              <nuxt-link
                :to="`/@${user.username}`"
                :title="`@${user.username}`">
                <avatar :avatar="user.content.avatar_image" />
              </nuxt-link>
            </li>
          </ul>
        </div>
      </template>
    </div>
    <div
      v-if="!viewOnly && user && !post.is_deleted"
      class="ml-auto mt-1">
      <div
        class="btn-group-vertical"
        role="group">
        <action-button
          ref="favorite"
          :resource="`/posts/${mainPost.id}/bookmark`"
          :icon="[['far', 'star'], ['fas', 'star']]"
          v-model="mainPost.you_bookmarked" />
        <action-button
          v-if="!me"
          ref="repost"
          :resource="`/posts/${mainPost.id}/repost`"
          v-model="mainPost.you_reposted"
          icon="retweet" />
      </div>
    </div>
  </div>
</template>

<script>
import ActionButton from '~/components/ActionButton'
import Avatar from '~/components/Avatar'
import Thumb from '~/components/Thumb'
import Sound from '~/components/Sound'
import Poll from '~/components/Poll'
import { mapGetters } from 'vuex'
import EntityText from '~/components/EntityText'
import {
  getImageURLs,
  getAudio,
  getCrosspostLink,
  getSpoiler,
  getLongpost,
  getChannelInvite
} from '~/assets/js/util'
import listItem from '~/assets/js/list-item'

export default {
  components: {
    ActionButton,
    Thumb,
    Sound,
    Avatar,
    EntityText,
    Poll
  },
  mixins: [listItem],
  dateKey: 'post.created_at',
  props: {
    post: {
      type: Object,
      required: true
    },
    viewOnly: {
      type: Boolean,
      default: false
    },
    detail: {
      type: Boolean,
      default: false
    },
    preview: {
      type: Boolean,
      default: false
    }
  },
  computed: {
    poll() {
      if (!this.mainPost.raw) return
      const raw = this.mainPost.raw.filter(
        item => item.type === 'io.pnut.core.poll-notice'
      )[0]
      if (!raw) return
      return raw.value
    },
    reactionUsers() {
      if (!this.detail || this.post.is_deleted) return []
      const users = this.mainPost.bookmarked_by.concat(
        this.mainPost.reposted_by
      )
      return (
        users
          // .slice(0, 10)
          .reduce((res, user) => {
            let exist = false
            for (let i = 0; res.length > i; i++) {
              if (res[i].id === user.id) {
                exist = true
                break
              }
            }
            if (!exist) {
              res.push(user)
            }
            return res
          }, [])
      )
    },
    thumbs() {
      return getImageURLs(this.mainPost)
    },
    clips() {
      return getAudio(this.mainPost)
    },
    crosspost() {
      return getCrosspostLink(this.mainPost)
    },
    spoiler() {
      return getSpoiler(this.mainPost)
    },
    longpost() {
      return getLongpost(this.mainPost)
    },
    channelInvite() {
      return getChannelInvite(this.mainPost)
    },
    me() {
      return this.user && this.user.id === this.post.user.id
    },
    mainPost() {
      return this.post.repost_of || this.post
    },
    permalink() {
      return `/@${this.mainPost.user.username}/posts/${this.mainPost.id}`
    },
    reply_permalink() {
      return `/posts/${this.mainPost.reply_to}`
    },
    revisions_permalink() {
      return `/posts/${this.mainPost.id}/revisions`
    },
    ...mapGetters(['user'])
  },
  methods: {
    replyModal() {
      if (this.mainPost.is_deleted) return
      this.$modal.show('post-modal', this.mainPost)
    },
    goPost() {
      this.$router.push(this.permalink)
    },
    favoriteToggle() {
      if (this.mainPost.is_deleted) return
      this.$refs.favorite.toggle()
    },
    repostToggle() {
      if (this.me || this.mainPost.is_deleted) return
      this.$refs.repost.toggle()
    },
    async removeModal() {
      if (!this.me || this.mainPost.is_deleted) return
      try {
        await this.$modal.show('remove-modal', this.post)
        this.remove()
      } catch (e) {}
    },
    async remove() {
      const { data: post } = await this.$axios.$delete(`/posts/${this.post.id}`)
      this.$toast.success('Deleted Post!')
      this.$emit('update:post', post)
    }
  }
}
</script>
<style scoped lang="scss">
@import '~assets/css/override';
@import '~bootstrap/scss/functions';
@import '~bootstrap/scss/variables';
@import '~bootstrap/scss/mixins';

.media-body {
  word-break: break-word;
}

footer {
  font-size: 0.85rem;
}

.media {
  .reply,
  .remove,
  .source,
  .crosspost-url {
    opacity: 0;
    transition: all 0.2s ease;
  }
  &:hover,
  &:focus {
    .reply,
    .remove,
    .source,
    .crosspost-url {
      opacity: 1;
    }
  }
}

.count {
  line-height: 1;
}

.iconSize {
  width: 48px;
  height: 48px;
  @include media-breakpoint-up(sm) {
    width: 64px;
    height: 64px;
  }
}

.deleted {
  opacity: 0.5;
}

.text-gray-dark {
  color: $gray;
}
</style>
