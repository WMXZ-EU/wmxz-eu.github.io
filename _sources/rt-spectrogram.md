# RT Spectrogram
The following real-time spectrogram is based in part on 
https://python-sounddevice.readthedocs.io/en/0.3.14/examples.html#plot-microphone-signal-s-in-real-time

The code takes the audio from the PC microphone and shows a plain spectrogram.
The purpose of this code is to demonstrate the cooperation of audio and animation callbacks via a common queue element.


    # RT Spectrogram
    import numpy as np
    import scipy.signal as signal

    import sounddevice as sd
    import queue

    import matplotlib.pyplot as plt
    from matplotlib import animation

    frame_rate=10
    ndat=512
    nfft=int(2*ndat)
    nf=1+int(nfft/2)
    ww= signal.windows.hann(nfft)

    tmp1=np.zeros((ndat,1))

    q=queue.Queue()
    def callback(indata, frames, time, status):
        global tmp1, tmp2
        if status:
            print(status)
        tmp2=np.concatenate((tmp1,indata))*ww
        tmp1=indata.copy()
        tmp3=np.abs(np.fft.rfft(tmp2,n=nfft,axis=0))**2
        tmp3[1:] *=2
        tmp3 /= nfft
        q.put(tmp3)

    acq = sd.InputStream(channels=1, blocksize=ndat, callback=callback)
    #-------------------------------------------------------------------
    fig=plt.figure("figure.figsize",[18.,6.7],facecolor="white")

    plotdata = np.zeros((nf,1000))
    im = plt.imshow(plotdata, 
                    extent=(0,ndat/44.1,  0,44.1/2), 
                    cmap="jet", aspect="auto", origin="lower", 
                    animated=True)
    im.set_clim(-80,0)

    CB=plt.colorbar(im,shrink=0.8, extend='both')

    ax=plt.gca()
    l, b, w, h = ax.get_position().bounds
    ll, bb, ww, hh = CB.ax.get_position().bounds
    #
    l=0.1; w=0.8
    ll=l+w+0.03; bb=b + 0.1*h; hh=h*0.8
    #
    ax.set_position([l, b, w, h])
    CB.ax.set_position([ll, bb, ww, hh])

    def animate(i):
        global plotdata
        while True:
            try:
                tmp4 = q.get_nowait()
            except queue.Empty:
                break

            plotdata = np.roll(plotdata, -1, axis=1)
            plotdata[:, -1:]=10*np.log10(tmp4+1e-6)

        im.set_array(plotdata)
        ax=plt.gca()
        ax.set_title('Time (s): ' + '{:.2f}'.format(i/frame_rate))
        return im,

    anim = animation.FuncAnimation(fig, animate, interval=1000/frame_rate)

    with acq:
        plt.show()
