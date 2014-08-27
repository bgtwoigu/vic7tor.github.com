android stagefrightʹ�õ���openmax il��һ�㣬û��ʹ��openmax al��һ�㡣

#1.OMXCodec::Create

        status_t err = omx->allocateNode(componentName, observer, &node);
        if (err == OK) {
            ALOGV("Successfully allocated OMX node '%s'", componentName);

            sp<OMXCodec> codec = new OMXCodec(
                    omx, node, quirks, flags,
                    createEncoder, mime, componentName,
                    source, nativeWindow);

            observer->setCodec(codec);

            err = codec->configureCodec(meta);

            if (err == OK) {
                if (!strcmp("OMX.Nvidia.mpeg2v.decode", componentName)) {
                    codec->mFlags |= kOnlySubmitOneInputBufferAtOneTime;
                }

                return codec;
            }

            ALOGV("Failed to configure codec '%s'", componentName);
        }

������omx -> IOMX��OMX : public BnOMX


##1.OMX::allocateNode
    OMXNodeInstance *instance = new OMXNodeInstance(this, observer);

    OMX_COMPONENTTYPE *handle;
    OMX_ERRORTYPE err = mMaster->makeComponentInstance(
            name, &OMXNodeInstance::kCallbacks,
            instance, &handle);

###makeComponentInstance
1.SoftOMXPlugin::makeComponentInstance���������ʵ�֣�

	SoftOMXPlugin::makeComponentInstance���� {
		        AString libName = "libstagefright_soft_";
        libName.append(kComponents[i].mLibNameSuffix);
        libName.append(".so");

        void *libHandle = dlopen(libName.c_str(), RTLD_NOW);

        if (libHandle == NULL) {
            ALOGE("unable to dlopen %s", libName.c_str());

            return OMX_ErrorComponentNotFound;
        }

        typedef SoftOMXComponent *(*CreateSoftOMXComponentFunc)(
                const char *, const OMX_CALLBACKTYPE *,
                OMX_PTR, OMX_COMPONENTTYPE **);

        CreateSoftOMXComponentFunc createSoftOMXComponent =
            (CreateSoftOMXComponentFunc)dlsym(
                    libHandle,
                    "_Z22createSoftOMXComponentPKcPK16OMX_CALLBACKTYPE"
                    "PvPP17OMX_COMPONENTTYPE");

        if (createSoftOMXComponent == NULL) {
            dlclose(libHandle);
            libHandle = NULL;

            return OMX_ErrorComponentNotFound;
        }
        sp<SoftOMXComponent> codec =
            (*createSoftOMXComponent)(name, callbacks, appData, component);
	}

SoftAAC2.cpp:

	android::SoftOMXComponent *createSoftOMXComponent(
        	const char *name, const OMX_CALLBACKTYPE *callbacks,
        	OMX_PTR appData, OMX_COMPONENTTYPE **component) {
    		return new android::SoftAAC2(name, callbacks, appData, component);
	}

##2.SoftOMXComponent
SoftAAC2.h�У�SoftAAC2������Ǽ̳�SimpleSoftOMXComponent�����ģ���useBuffer��allocateBuffer��fillThisBuffer��Щ���Ѿ���SimpleSoftOMXComponent��ʵ�֣���SoftAAC2.h��ֻ��Ҫʵ�ָ��ٵ�������ˡ�

##3.Ӳ��ʵ��
QComOMXPlugin.h (hardware\qcom\media\libstagefrighthw)

QComOMXPlugin::QComOMXPlugin()
    : mLibHandle(dlopen("libOmxCore.so", RTLD_NOW)),
      mInit(NULL),
      mDeinit(NULL),
      mComponentNameEnum(NULL),
      mGetHandle(NULL),
      mFreeHandle(NULL),
      mGetRolesOfComponentHandle(NULL) {
    if (mLibHandle != NULL) {
        mInit = (InitFunc)dlsym(mLibHandle, "OMX_Init");
        mDeinit = (DeinitFunc)dlsym(mLibHandle, "OMX_Deinit");

        mComponentNameEnum =
            (ComponentNameEnumFunc)dlsym(mLibHandle, "OMX_ComponentNameEnum");

        mGetHandle = (GetHandleFunc)dlsym(mLibHandle, "OMX_GetHandle");
        mFreeHandle = (FreeHandleFunc)dlsym(mLibHandle, "OMX_FreeHandle");

        mGetRolesOfComponentHandle =
            (GetRolesOfComponentFunc)dlsym(
                    mLibHandle, "OMX_GetRolesOfComponent");

        (*mInit)();
    }
}

OMX_ERRORTYPE QComOMXPlugin::makeComponentInstance(
        const char *name,
        const OMX_CALLBACKTYPE *callbacks,
        OMX_PTR appData,
        OMX_COMPONENTTYPE **component) {
    if (mLibHandle == NULL) {
        return OMX_ErrorUndefined;
    }

    return (*mGetHandle)(
            reinterpret_cast<OMX_HANDLETYPE *>(component),
            const_cast<char *>(name),
            appData, const_cast<OMX_CALLBACKTYPE *>(callbacks));
}

#2.��������
makeComponentInstance�ķ���ֵ��������OMXNodeInstance::setHandle�Ĳ�����

status_t OMXNodeInstance::sendCommand(
        OMX_COMMANDTYPE cmd, OMX_S32 param) {
    Mutex::Autolock autoLock(mLock);

    OMX_ERRORTYPE err = OMX_SendCommand(mHandle, cmd, param, NULL);
    return StatusFromOMXError(err);
}

OMX_SendCommand��ʵ�֣�
OMX_Core.h (hardware\qcom\media\mm-core\inc)

#define OMX_SendCommand(                                    \
         hComponent,                                        \
         Cmd,                                               \
         nParam,                                            \
         pCmdData)                                          \
     ((OMX_COMPONENTTYPE*)hComponent)->SendCommand(         \
         hComponent,                                        \
         Cmd,                                               \
         nParam,                                            \
         pCmdData)                          /* Macro End */

Ψһʵ�ֵ���OMX_GetHandle��

##3.OMX_COMPONENTTYPE
��������ʵ����Ҳ�᷵��һ����������������openmax il�ж���ģ���OMX_SendCommand����ʹ�õ�������ṹ��ĵĺ�����

����������ʵ�֣�����SoftOMXComponent::SoftOMXComponent�г�ʼ���ģ�

	mComponent->SendCommand = SendCommandWrapper;

OMX_ERRORTYPE SoftOMXComponent::SendCommandWrapper(
        OMX_HANDLETYPE component,
        OMX_COMMANDTYPE cmd,
        OMX_U32 param,
        OMX_PTR data) {
    SoftOMXComponent *me =
        (SoftOMXComponent *)
            ((OMX_COMPONENTTYPE *)component)->pComponentPrivate;

    return me->sendCommand(cmd, param, data);
}

#СС���ܽ�
OMX::allocateNode():

1.�õ�OMX_COMPONENTTYPE

mMaster->makeComponentInstance 

2.��һ��ID����¼binderʱʹ�õ�id������OMX_COMPONENTTYPEʵ��

    *node = makeNodeID(instance);
    mDispatchers.add(*node, new CallbackDispatcher(instance));

3.����

status_t OMX::sendCommand(
        node_id node, OMX_COMMANDTYPE cmd, OMX_S32 param) {
    return findInstance(node)->sendCommand(cmd, param);
}

    instance->setHandle(*node, handle);

    mLiveNodes.add(observer->asBinder(), instance);

# OMXMaster::makeComponentInstance

mPluginByComponentName����OMXMaster::addPlugin���ӵ�

addPlugin�ĵ��ã�
OMXMaster::OMXMaster():

    addVendorPlugin();
    addPlugin(new SoftOMXPlugin);

#Ӧ�����
AwesomePlayer::initAudioDecoder��OMXCodec::Create�е�matchComponetNameΪNULL��ֻ����isLPAPlaybackʱ��һ�¡�

##OMXCodec::findMatchingCodecs
����б���MediaCodecList�������ʵ�֣���������device/qcom/common/media/media_codecs.xml����ļ����г�����codec

SoftOMXPlugin��MediaCodecList����ϵ��
static const struct {
    const char *mName;
    const char *mLibNameSuffix;
    const char *mRole;

} kComponents[] = {
    { "OMX.google.aac.decoder", "aacdec", "audio_decoder.aac" },
    { "OMX.google.aac.encoder", "aacenc", "audio_encoder.aac" },
    { "OMX.google.amrnb.decoder", "amrdec", "audio_decoder.amrnb" },
    { "OMX.google.amrnb.encoder", "amrnbenc", "audio_encoder.amrnb" },
    { "OMX.google.amrwb.decoder", "amrdec", "audio_decoder.amrwb" },
    { "OMX.google.amrwb.encoder", "amrwbenc", "audio_encoder.amrwb" },
    { "OMX.google.h264.decoder", "h264dec", "video_decoder.avc" },
    { "OMX.google.h264.encoder", "h264enc", "video_encoder.avc" },
    ...
};

�ڶ�����Ա����libstagefright_soft_aacdec.so�ļ����е���ʲô�ˡ�

##mime:
1.media_codecs.xml:

	<MediaCodec name="OMX.qcom.audio.decoder.wmaLossLess" type="audio/x-ms-wma" >

2.OMXCodec::findMatchingCodecs:

	list->findCodecByType(mime, createEncoder, index);

3.DataSource::Sniff

	DataSource::RegisterDefaultSniffers() {
		RegisterSniffer(SniffMPEG4);
		...
	}

SniffMPEG4ʵ����MPEG4Extractor.cpp��

void DataSource::RegisterSniffer(SnifferFunc func) {
    Mutex::Autolock autoLock(gSnifferMutex);

    for (List<SnifferFunc>::iterator it = gSniffers.begin();
         it != gSniffers.end(); ++it) {
        if (*it == func) {
            return;
        }
    }

    gSniffers.push_back(func);
}

	MediaExtractor::Create() {
		...
		source->sniff(&tmp, &confidence, &meta)
		...
		new MPEG4Extractor(source);
	}

MIME�ඨ��MediaDefs.cpp:

	const char *MEDIA_MIMETYPE_CONTAINER_MPEG4 = "video/mp4";

##�����������Դ
Awesomeplay::setDataSource()
��MediaExtractor::getTrack���DataSource��Ȼ��͸�OMX��Щ����������
