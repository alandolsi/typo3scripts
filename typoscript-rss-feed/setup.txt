# Create rss feeds only with typoscript
# Author: Christian Blechert (christian@blechert.name)
# Homepage: https://github.com/perryflynn/typo3scripts/
# Original Source: http://www.thefox.ch/typo/code-schnipsel/typoscript/wie-mache-ich/ein-rss-feed-in-typo3-erstellen/

# See Readme.md for more information!

config.disablePrefixComment = 1
tt_content.stdWrap.innerWrap >

# rss feed template
temp.rssTemplate = COA
temp.rssTemplate {
 
  1 = LOAD_REGISTER
  1 {
  # config
    # store id of page with sitemap from constants to register
    pageid = {$rsstemplate.pageid}
    # store id of content element from constants to register
    contentid = {$rsstemplate.contentid}
 
    # Set other settings from constants
    title = {$rsstemplate.meta.title}
    description = {$rsstemplate.meta.description}
    copyright = {$rsstemplate.meta.copyright}
    language = {$rsstemplate.meta.language}
 
    image_width = {$rsstemplate.meta.image_width}
    image_height = {$rsstemplate.meta.image_height}
    image_description < .description
    image_url = {$rsstemplate.meta.image_url}
 
    item_author = {$rsstemplate.meta.author}
    item_author_email = {$rsstemplate.meta.author_email}
  }
 
  10 = TEMPLATE
  10 {
 
    # xml template
    template = FILE
    template.file = fileadmin/template/blog/rss.xml
 
    marks {
 
      RSS_title = TEXT
      RSS_title.data = register:title
 
      RSS_description = TEXT
      RSS_description.data = register:description
 
      RSS_atom_link = COA
      RSS_atom_link {
        1 = TEXT
        1.data = getenv : HTTP_HOST
        1.rawUrlEncode = 0
        1.wrap = http://|
        2.wrap = |
        2 = TEXT
        2.data = getenv : REQUEST_URI
        2.rawUrlEncode = 0
      }
      
      RSS_copyright = TEXT
      RSS_copyright.data = register:copyright
 
      RSS_language = TEXT
      RSS_language.data = register:language
 
      RSS_image_url = TEXT
      RSS_image_url.value = {getIndpEnv:TYPO3_SITE_URL}{register:image_url}
      RSS_image_url.insertData = 1
      
      RSS_image_link = TEXT
      RSS_image_link.value = {getIndpEnv:TYPO3_SITE_URL}
      RSS_image_link.insertData = 1
 
      RSS_link < .RSS_image_link
 
      RSS_image_width = TEXT
      RSS_image_width.data = register:image_width
 
      RSS_image_height = TEXT
      RSS_image_height.data = register:image_height
 
      RSS_image_description = TEXT
      RSS_image_description.data = register:image_description
 
      RSS_pubDate = TEXT
      RSS_pubDate {
        data = register:SYS_LASTCHANGED
        date = r
      }
 
      RSS_lastBuildDate = TEXT
      RSS_lastBuildDate {
        data = register:SYS_LASTCHANGED
        date = r
      }
 
      RSS_ITEMS = COA
      RSS_ITEMS {
        1 = LOAD_REGISTER
        1 {
          # read sitemap content element
          pages.cObject = CONTENT
          pages.cObject {
            table = tt_content
 
            select {
              pidInList.dataWrap = {register:pageid}
              where.dataWrap = uid= {register:contentid}
              languageField = sys_language_uid
              insertData = 1
            }
 
            renderObj = TEXT
            renderObj {
              field = pages
            }
          }
        }
 
        # create the xml structure with a hmenu
        10 = HMENU
        10 {
          # exclude typo3_blog category pages
          excludeDoktypes = 73
          special = updated
          special.value.data = register:pages
          special {
            # Newest by edit date
            mode = tstamp
            # Follow tree to a depth of x
            depth = 5
            # Max x entires in feed
            limit = 25
          }
          wrap = |
          1 = TMENU
          1 {
            target = {$PAGE_TARGET}
            noBlur = 1
 
            NO {
              doNotLinkIt = 1
              doNotShowLink = 1
              stdWrap2 {
                # Build fields for rss item
                cObject = COA
                cObject {
                  1 = LOAD_REGISTER
                  1 {
                    title {
                      field = title
                      htmlSpecialChars = 1
                    }
 
                    subtitle {
                      field = subtitle
                      noTrimWrap = | : | |
                      required = 1
                      htmlSpecialChars = 1
                    }
 
                    link_and_guid {
                      typolink {
                        parameter.field = uid
                        returnLast = url
                      }
                      wrap = {getIndpEnv:TYPO3_SITE_URL}|
                      insertData = 1
                    }
 
                    guid.cObject = TEXT
                    guid.cObject {
                      value = {getIndpEnv:TYPO3_SITE_URL}?id={field:uid}
                      insertData = 1
                    }
 
                    page_author_email {
                      data = field:author_email // register:item_author_email
                    }
                  }
 
                  10 = TEXT
                  10 {
                    data = field:abstract // field:description
                    wrap = <description><![CDATA[|]]></description>
                    required = 1
                    htmlSpecialChars = 1
                    # output kosmetik
                    prepend = TEXT
                    prepend.char = 10
                    append = TEXT
                    append.char = 10
                  }
 
                  15 = TEXT
                  15 {
                    data = field:author // register:item_author
                    wrap = <author>{register:page_author_email} (|)</author>
                    insertData = 1
                    required = 1
                    # output kosmetik
                    append = TEXT
                    append.char = 10
                  }
 
                  40 = TEXT
                  40 {
                    wrap = <pubDate>|</pubDate>
                    data = field:crdate
                    date = r
                    # output kosmetik
                    append = TEXT
                    append.char = 10
                  }
                  
                  50 = CONTENT
                  50 {
                    table = tt_content
                    #wrap = <content:encoded><![CDATA[|]]></content:encoded>
                    wrap = <description><![CDATA[|]]></description>
                    stdWrap.replacement {
                      10 {
                        search = #<iframe.*?>.*?</iframe>#i
                        replace = <span style="font-size:16px; color:red; font-weight:bold;">Hier ist eigentlich ein iFrame.<br>Besuche den Blog um ihn zu sehen!</span>
                        useRegExp = 1
                      },
                      11 {
                        search = #(href|src)="\.\./#i
                        replace = TEXT
                        # !!!! CHANGEME !!!! #
                        replace = \1="http://anwendungsentwickler.ws/
                        useRegExp = 1
                      },
                      12 {
                        search = #(href|src)="/#i
                        replace = TEXT
                        # !!!! CHANGEME !!!! #
                        replace = \1="http://anwendungsentwickler.ws/
                        useRegExp = 1
                      },
                      13 {
                        search = #(href|src)="(index|uploads|typo3temp)#i
                        replace = TEXT
                        # !!!! CHANGEME !!!! #
                        replace = \1="http://anwendungsentwickler.ws/\2
                        useRegExp = 1
                      },
                      20 {
                        search = |
                        replace = &#124;
                      }
                    }
                    select {
                      pidInList.dataWrap = {field:uid}
                      orderBy = crdate
                    }
                  }
                  
                }
              }
 
              allWrap (
              <item>
                <title>{register:title}{register:subtitle}</title>
                <link>{register:link_and_guid}</link>
                <guid>{register:guid}</guid>
                |
              </item>
              )
              allWrap.insertData = 1
            }
 
          }
        }
      }
    }
 
  }
}


# page template
rss200 = PAGE
rss200 {
  config {
  
    # enable realurl
    baseURL = {getIndpEnv:TYPO3_SITE_URL}
    tx_realurl_enable = 1
    
    # disable all html stuff
    disableAllHeaderCode = 1
    additionalHeaders = Content-type:text/xml;charset=utf-8
    linkVars = L,debug
    no_cache = 1
    xhtml_cleaning = 0
    admPanel = 0
  }
 
  # template einbinden
  10 < temp.rssTemplate
}


#EOF
