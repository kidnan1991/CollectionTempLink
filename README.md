# CollectionTempLink
http://stackoverflow.com/questions/41259575/why-didnt-have-data-when-the-first-enter-my-activity-it-make-up-viewpager-fra

https://www.nihongo-pro.com/free-jlpt-n5-quizzes

// Sample for crawler data
<pre>
package tsb.trinhnx.download;

import java.io.File;
import java.io.FileWriter;
import java.net.Authenticator;
import java.net.InetSocketAddress;
import java.net.PasswordAuthentication;
import java.net.Proxy;
import java.util.ArrayList;
import java.util.List;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

import org.jsoup.Connection;
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;

/**
 * @author callmeanonymous
 * Require : jsoup
 * TODO: Thím nào pro regex thì update hộ em, em mù tịt cái này.
 *
 */
public class VozCrawler {
    private static final String parentThread = "https://vozforums.com/showthread.php?t=5260246";
    private static final String userAccount = "vozforums.com/member.php?u=(.*?)";
    /**
     * Pattern valid in 1 Nick - 2 - 3
     * First process
     */
    private static final String NICK_PATTERN_1 = "Nick(.*?)2";
    /**
     * Patter valid in Nick - Lý do bị ban...
     * Second process
     */
    private static final String NICK_PATTERN_2 = "Nick(.*?)Lý";
    /**
     * Patter valid in Nick - Lí do bị ban...
     * Third process
     * Well, we are voz :lol:
     */
    private static final String NICK_PATTERN_3 = "Nick(.*?)Lí";

    private static final String NICK_PATTERN_4 = "nick(.*?)2";

    protected static final Pattern[] pattern = new Pattern[] { Pattern.compile(NICK_PATTERN_1),
            Pattern.compile(NICK_PATTERN_2), Pattern.compile(NICK_PATTERN_3) };

    public static void main(String[] args) throws Exception {

        final int defaultStartPage = 151;
        final int defaultEndPage = 170;
        final String urlFormat = "%s&page=%d";
        final String proxyHost = "proxyxxxxxx";
        final int proxyPort = 11111;
        final String proxyUser = "xxxxxx";
        final String proxyPassword = "xxxxxx";
        final Proxy httpProxy = new Proxy(Proxy.Type.HTTP,
                InetSocketAddress.createUnresolved(proxyHost, proxyPort));

        Authenticator.setDefault(new Authenticator() {
            @Override
            protected PasswordAuthentication getPasswordAuthentication() {
                return new PasswordAuthentication(proxyUser, proxyPassword.toCharArray());
            }
        });
        System.out.println("Start at : " + System.currentTimeMillis());
        final List<VozAccount> vozAccount = new ArrayList<>();
        for (int i = defaultStartPage; i < defaultEndPage; i++) {
            final int threadPage = i;
            final String threadLink = String.format(urlFormat, parentThread, threadPage);
            Connection conn = Jsoup.connect(threadLink);
            conn.proxy(httpProxy);
            conn.timeout(5000);
            Document doc = conn.get();
            Elements elements = doc.select(".tborder.voz-postbit");
            elements.forEach(element -> {
                final Element postId = element.getElementsByTag("strong").get(0);
                final Element postText = element.getElementsByClass("voz-post-message").get(0);
                final String id = postId.text();
                final String postMsg = postText.text();
                System.out.println("Message : " + postMsg);
                if (id.isEmpty() || !validPostMessage(postMsg)) {
                    System.err.println("Error parsing: " + postMsg);
                } else {
                    String name = refinedUserName(parseUserFromPost(postMsg, pattern));
                    if (name != null) {
                        vozAccount.add(new VozAccount(Integer.parseInt(id), name));
                    }
                }
            });
        }
        File outPut = new File("D:\\temp.txt");
        writeListToFile(vozAccount, outPut);
        final int total = vozAccount.size();
        System.out.println(vozAccount.get(0).getPostId() + vozAccount.get(0).getNickName());
        System.out.println(
                vozAccount.get(total - 1).getPostId() + vozAccount.get(total - 1).getNickName());
        System.out.println("Finish at : " + System.currentTimeMillis());
    }

    /**
     * @param name
     * @return
     * boolean
     */
    protected static boolean validUserName(String name) {
        if (name == null || name.isEmpty())
            return false;
        return !name.trim().contains(" ");
    }

    protected static boolean validPostMessage(final String postMsg) {
        return postMsg != null && !postMsg.trim().isEmpty();
    }

    /**
     * Get the user from post message
     * @param postMsg
     * @return
     * String
     */
    protected static String parseUserFromPost(final String postMsg, Pattern... patternList) {
        final String preprocess = postMsg.trim();
        for (Pattern pattern : patternList) {
            Matcher matcher = pattern.matcher(preprocess);
            while (matcher.find()) {
                return matcher.group(1);
            }
        }
        return null;
    }

    /**
     * Remove the space / : / ! character from parsed name
     * @param userName
     * @return null if invalid input (null)
     * String
     */
    protected static String refinedUserName(final String userName) {
        if (userName == null || userName.isEmpty()) {
            return null;
        }
        if (userName.contains("member.php?u")) {
            System.err.println("Invalid format " + userName);
            return null;
        }
        return userName.replace(":", "").trim();
    }

    /**
     * Get user from URL link
     * @param url
     * @return
     * String
     */
    protected static String parseUserFromUrl(final String url) {
        //        Matcher matcher = Pattern.compile(userAccount).matcher(url);
        //        if (matcher.find()) {
        //            return matcher.group(1);
        //        }
        if (url.contains("member.php"))
            return null;
        return url;
    }

    protected static void writeListToFile(List<? extends Object> data, final File outDirectory) {
        FileWriter writer;
        try {
            writer = new FileWriter(outDirectory, false);
            for (Object rawData : data) {
                writer.write(rawData.toString());
                writer.append(System.lineSeparator());
            }
            writer.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    private static final class VozAccount {
        private int postId;
        private String nickName;

        public int getPostId() {
            return postId;
        }

        public void setPostId(int postId) {
            this.postId = postId;
        }

        public String getNickName() {
            return nickName;
        }

        public void setNickName(String nickName) {
            this.nickName = nickName;
        }

        /**
         * @param postId
         * @param nickName
         */
        public VozAccount(int postId, String nickName) {
            super();
            this.postId = postId;
            this.nickName = nickName;
        }

        @Override
        public String toString() {
            return this.nickName;
        }
    }
}
</pre>
