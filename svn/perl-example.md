
# svn post-commit example

filename : post-commit
  
require perl module :   

```sh
yum install perl-libwww-perl perl-LWP-Protocol-https perl-URI
```

```perl
#!/usr/bin/env perl

use warnings;
use strict;

use HTTP::Request::Common qw(POST);
use HTTP::Status qw(is_client_error);
use LWP::UserAgent;
use JSON;

# 한글 사용을 위해 추가
use Encode qw(decode_utf8);

my $opt_domain = "example.slack.com";
my $opt_token = "{토큰}";

# 한글 사용을 위해 변경
my $log = `/usr/bin/svnlook log -r $ARGV[1] $ARGV[0]`;
$log = decode_utf8($log);

#my $log = `/usr/bin/svnlook log -r $ARGV[1] $ARGV[0]`;
my $files = `/usr/bin/svnlook changed -r $ARGV[1] $ARGV[0]`;
my $who = `/usr/bin/svnlook author -r $ARGV[1] $ARGV[0]`;
my $url = "http://svn.dev/repos/kcx/?op=revision&rev=$ARGV[1]"; # optionally set this to the url of your internal commit browser. Ex: http://svnserver/wsvn/main/?op=revision&rev=$ARGV[1]
chomp $who;

my $payload = {
        'revision'      => $files.'['.$ARGV[1].']',
        'url'           => $url,
        'author'        => $who,
        'log'           => $log,
};

my $ua = LWP::UserAgent->new;
$ua->timeout(15);

my $req = POST( "https://${opt_domain}/services/hooks/subversion?token=${opt_token}", ['payload' => encode_json($payload)] );
my $s = $req->as_string;
print STDERR "Request:\n$s\n";

my $resp = $ua->request($req);
$s = $resp->as_string;
print STDERR "Response:\n$s\n";
```