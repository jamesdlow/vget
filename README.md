# vget

Video Download Library.

Support for Youtube, Vimeo. Multithread downloads. Based on multithread download library https://github.com/axet/wget.

Pure java!

Inspired by http://sourceforge.net/projects/ytd2/.

Code taken from ytd2 and completely rewritten to support more web sites.

Good examples here:
  https://github.com/pculture/miro/blob/master/tv/lib/flashscraper.py

## Exceptions

Here are a two kind of exceptions.

1) Fatal exception. all RuntimeException's
  We shall stop application

2) DownloadError (extends RuntimeException)
  We unable to process following url and shall stop to download it

## Example Direct Download

    package com.github.axet.vget;
    
    import java.io.File;
    import java.net.URL;
    
    public class DirectDownload {
    
        public static void main(String[] args) {
            try {
                // ex: http://www.youtube.com/watch?v=Nj6PFaDmp6c
                String url = args[0];
                // ex: "/Users/axet/Downloads"
                String path = args[1];
                VGet v = new VGet(new URL(url), new File(path));
                v.download();
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        }
    
    }

## Example Application Managed Download

    package com.github.axet.vget;
    
    import java.io.File;
    import java.net.URL;
    import java.util.List;
    import java.util.concurrent.atomic.AtomicBoolean;
    
    import com.github.axet.vget.info.VGetParser;
    import com.github.axet.vget.info.VideoInfo;
    import com.github.axet.vget.vhs.VimeoInfo;
    import com.github.axet.vget.vhs.YoutubeInfo;
    import com.github.axet.wget.info.DownloadInfo;
    import com.github.axet.wget.info.DownloadInfo.Part;
    import com.github.axet.wget.info.DownloadInfo.Part.States;
    
    public class AppManagedDownload {
    
        VideoInfo info;
        long last;
    
        public void run(String url, File path) {
            try {
                AtomicBoolean stop = new AtomicBoolean(false);
                Runnable notify = new Runnable() {
                    @Override
                    public void run() {
                        VideoInfo i1 = info;
                        DownloadInfo i2 = i1.getInfo();
    
                        // notify app or save download state
                        // you can extract information from DownloadInfo info;
                        switch (i1.getState()) {
                        case EXTRACTING:
                        case EXTRACTING_DONE:
                        case DONE:
                            if (i1 instanceof YoutubeInfo) {
                                YoutubeInfo i = (YoutubeInfo) i1;
                                System.out.println(i1.getState() + " " + i.getVideoQuality());
                            } else if (i1 instanceof VimeoInfo) {
                                VimeoInfo i = (VimeoInfo) i1;
                                System.out.println(i1.getState() + " " + i.getVideoQuality());
                            } else {
                                System.out.println("downloading unknown quality");
                            }
                            break;
                        case RETRYING:
                            System.out.println(i1.getState() + " " + i1.getDelay());
                            break;
                        case DOWNLOADING:
                            long now = System.currentTimeMillis();
                            if (now - 1000 > last) {
                                last = now;
    
                                String parts = "";
    
                                List<Part> pp = i2.getParts();
                                if (pp != null) {
                                    // multipart download
                                    for (Part p : pp) {
                                        if (p.getState().equals(States.DOWNLOADING)) {
                                            parts += String.format("Part#%d(%.2f) ", p.getNumber(), p.getCount()
                                                    / (float) p.getLength());
                                        }
                                    }
                                }
    
                                System.out.println(String.format("%s %.2f %s", i1.getState(),
                                        i2.getCount() / (float) i2.getLength(), parts));
                            }
                            break;
                        default:
                            break;
                        }
                    }
                };
    
                URL web = new URL(url);
    
                // [OPTIONAL] limit maximum quality, or do not call this function if
                // you wish maximum quality available.
                //
                // if youtube does not have video with requested quality, program
                // will raise en exception.
                VGetParser user = null;
    
                // create proper html parser depends on url
                user = VGet.parser(web);
    
                // download maximum video quality from youtube
                // user = new YouTubeQParser(YoutubeQuality.p480);
    
                // download mp4 format only, fail if non exist
                // user = new YouTubeMPGParser();
    
                // create proper videoinfo to keep specific video information
                info = user.info(web);
    
                VGet v = new VGet(info, path);
    
                // [OPTIONAL] call v.extract() only if you d like to get video title
                // or download url link
                // before start download. or just skip it.
                v.extract(user, stop, notify);
    
                System.out.println("Title: " + info.getTitle());
                System.out.println("Download URL: " + info.getInfo().getSource());
    
                v.download(user, stop, notify);
            } catch (RuntimeException e) {
                throw e;
            } catch (Exception e) {
                throw new RuntimeException(e);
            }
        }
    
        public static void main(String[] args) {
            AppManagedDownload e = new AppManagedDownload();
            // ex: http://www.youtube.com/watch?v=Nj6PFaDmp6c
            String url = args[0];
            // ex: /Users/axet/Downloads/
            String path = args[1];
            e.run(url, new File(path));
        }
    }
    
## Central Maven Repo

    <dependency>
      <groupId>com.github.axet</groupId>
      <artifactId>vget</artifactId>
      <version>1.1.23</version>
    </dependency>

