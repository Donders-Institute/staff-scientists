**********************************************************
What to do with the 20 Hz + harmonics artifact in the MEG?
**********************************************************

Since its introduction in July, there is an artifact in the MEG signals, caused by the light camera in the MSR. Data that has been collected between early July 2018 and <some-date-in-the-future-once-it-is resolved> is most likely affected.

What's wrong?
=============

The Raspberry pi operated camera - when switched on - produces an artifact at 20 Hz and its harmonics, which produces a series of artifact lines in the powerspectra of the MEG signals. These lines are sharp when there is no (or modest) fluctuations in light picked up by the camera. However, the spectral lines can be appreciably broadened when the projector is projectings stuff on the screen. This suggests that there is some amplitude modulation of the artifact.

How to check how bad it is in my data?
======================================

The easiest way to check whether you are affected is to compute a powerspectrum of the MEG signals with a sufficiently high spectral resolution.

.. code-block:: matlab

  cfg = [];
  cfg.dataset = fullfile(<path-to-data>, <ds-dirname>);
  hdr = ft_read_header(cfg.dataset);
  
  % quick and dirty 5 second chunks
  N = hdr.nSamples*hdr.nTrials;
  trl = [1:3000:(N-5999);6000:3000:N]';
  trl(:,3) = 0;
  
  % let's go for about 10 minutes of data, or less
  cfg.trl = trl(1:min(size(trl,1),12*10),:);
  cfg.channel = {'MEG' 'MEGREF'};
  cfg.demean  = 'yes';
  cfg.continuous = 'yes';
  data = ft_preprocessing(cfg);
  
  % this may be needed, because there may be some chunks of bad data in this recording, that occlude the artifact
  S = [];
  for kk = 1:numel(data.trial)
    S(:,kk) = std(data.trial{kk},[],2);
  end
  S = S./std(S,[],2);
  
  cfg = [];
  cfg.trials = find(mean(S)<2);
  data = ft_selectdata(cfg, data);
    
  cfg = [];
  cfg.method  = 'mtmfft';
  cfg.output  = 'pow';
  cfg.foilim  = [0 200];
  cfg.taper   = 'hanning';
  cfg.channel = 'MEG';
  freq      = ft_freqanalysis(cfg, data);
  
  figure;
  plot(freq.freq,mean(log10(freq.powspctrm))); title('powerspectrum','interpreter','none'); ylim([-30 -27]);
  
