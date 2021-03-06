
# svn post-commit example

filename : post-commit
  
require perl module :   

```sh
# CentOS 7
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

# encoding 
use Encode qw(decode_utf8);

my $opt_domain = "{your slack domain}";
my $opt_token = "{your token}";

# charset utf 8
#my $log = `/usr/bin/svnlook log -r $ARGV[1] $ARGV[0]`;
my $log=qx|export LC_CTYPE="en_US.utf8"; /usr/bin/svnlook log -r $ARGV[1] $ARGV[0]|;
$log = decode_utf8($log);

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
