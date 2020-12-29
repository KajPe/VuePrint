<template>
    <div>
        <div @click="print">
            <slot></slot>
        </div>

        <iframe
            ref="vuePrintIframe"
            class="iframe"
            src="about:blank"
            loading="eager"
            @load="setLoaded"
        ></iframe>
    </div>
</template>

<style scoped>
.iframe {
    border: 0;
    position: absolute;
    width: 0;
    height: 0;
    left: -100px;
    top: -100px;
}
</style>

<script>
import Vue from 'vue'

const TIME_OUT_ADD = 100

export default {
    /**
     * https://github.com/KajPe/VuePrint
     * License: MIT
     */

    /**
     * Props
     *  Also has the following events
     *      @onAfterPrint
     *          after printing, when printer selection dialog opens
     *      @onLoading (state)
     *          state is True during processing, otherwise False
     *          Can be used to show a loading dialog
     *      @onTimeoutError
     *          Timeout on frame html processing
     *      @onLoadError
     *          Load error (error in callback)
     */
    props: {
        callback: Function,
        timeout: {
            type: Number,
            default: 5000
        },
    },
    
    /**
     * Data
     */
    data() {
        return {
            loading: false,
            loaded: false,
        }
    },

    /**
     * Methods
     */
    methods: {
        /**
         * Set/Clear iFrame content
         */
        setIFrameContent(iframe, printContent = null) {
            iframe.document.open('text/html', 'replace')
            if (printContent) {
                iframe.document.write(printContent)
            }
            iframe.document.close()
        },

        /**
         * After print; remove event listener and emit onAfterPrint event
         */
        afterPrint(event) {
            let iframe = this.$refs.vuePrintIframe.contentWindow
            this.setIFrameContent(iframe)
            iframe.removeEventListener('afterprint', this.afterPrint)
            this.$emit('onAfterPrint')
        },

        /**
         * Content loaded
         */
        setLoaded() {
        	this.loaded = true
        },

        /**
         * Promise: wait for frame to get loaded (or timeout)
         */
        waitForFrameLoad(timeout_ms) {
            return new Promise( (resolve, reject) => {
                var check = () => {
                    if (this.loaded) {
                        // Loaded
                        resolve()
                    } else if ((timeout_ms -= TIME_OUT_ADD) < 0) {
                        // Timeout
                        reject()
                    } else {
                        // Still waiting
                        setTimeout(check, TIME_OUT_ADD)
                    }
                }
                setTimeout(check, TIME_OUT_ADD)
            })
        },

        /**
         * Set loading state and emit onLoading event
         */
        setLoading(state) {
            this.loading = state
            this.$emit('onLoading', state)
        },

        /**
         * Check if value is promise
         */
        isPromise(value) {
            return Boolean(value && typeof value.then === 'function')
        },

        /**
         * Loader: Returns promise
         */
        loader() {
            return new Promise( (resolve, reject) => {
                if (this.callback) {
                    let result = this.callback()
                    if (this.isPromise(result)) {
                        // Is promise (for example axios)
                        result
                        .then( (response) => {
                            resolve(response.data)
                        })
                        .catch( () => {
                            // Problems
                            reject()
                        })
                    } else {
                        // String, returns promise
                        resolve(result)
                    }
                } else {
                    // No callback, return reject
                    reject()
                }
            })
        },

        /**
         * Print
         */
        print() {
            this.setLoading(true)

            let iframe = this.$refs.vuePrintIframe.contentWindow

            this.loader()
            .then( (printContent) => {
                this.loaded = false
                this.setIFrameContent(iframe, printContent)

                // Wait for iFrame to be fully loaded
                this.waitForFrameLoad(this.timeout)
                .then( () => {
                    iframe.addEventListener('afterprint', this.afterPrint)

                    this.setLoading(false)

                    // nextTick and setTimeout so that Vue can render the loading -dialog as hidden
                    this.$nextTick( () => {
                        setTimeout( () => {
                            iframe.focus()
                            iframe.print()
                        }, 100)
                    })
                })
                .catch( () => {
                    this.setIFrameContent(iframe)
                    this.setLoading(false)
                    this.$emit('onTimeoutError')
                })
            })
            .catch( () => {
                this.setIFrameContent(iframe)
                this.setLoading(false)
                this.$emit('onLoadError')
            })
        },
    }
}
</script>